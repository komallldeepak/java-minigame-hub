package com.student.studentregistration;

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.time.LocalDate;
import java.util.Random;

import com.mongodb.client.*;
import com.mongodb.client.model.UpdateOptions;
import org.bson.Document;

public class PiggyGame extends JFrame {

    CardLayout card = new CardLayout();
    JPanel mainPanel = new JPanel(card);

    String username;
    int xp = 0;

    MongoCollection<Document> collection;

    public PiggyGame() {

        MongoClient client = MongoClients.create("mongodb://localhost:27017");
        MongoDatabase db = client.getDatabase("school");
        collection = db.getCollection("players");

        showLogin();

        add(mainPanel);
        setSize(800, 500);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setVisible(true);
    }

    // ================= LOGIN =================
    void showLogin() {

        JPanel login = new JPanel() {
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                Graphics2D g2 = (Graphics2D) g;
                GradientPaint gp = new GradientPaint(0, 0, Color.PINK, getWidth(), getHeight(), Color.CYAN);
                g2.setPaint(gp);
                g2.fillRect(0, 0, getWidth(), getHeight());
            }
        };

        login.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();

        JLabel title = new JLabel("🎮 PIGGY GAME HUB");
        title.setFont(new Font("Arial", Font.BOLD, 32));
        title.setForeground(Color.WHITE);

        // Added fun subtitle
        JLabel subtitle = new JLabel("Level Up Your Skills • Collect XP");
        subtitle.setFont(new Font("Arial", Font.ITALIC, 16));
        subtitle.setForeground(new Color(255, 255, 255, 200));

        JTextField userField = new JTextField(15);
        userField.setFont(new Font("Arial", Font.PLAIN, 18));

        JButton btn = new JButton("🚀 ENTER THE HUB");
        btn.setFont(new Font("Arial", Font.BOLD, 18));
        btn.setBackground(new Color(0, 0, 0));
        btn.setForeground(Color.WHITE);
        btn.setPreferredSize(new Dimension(200, 50));

        btn.addMouseListener(new MouseAdapter() {
            public void mouseEntered(MouseEvent e) { 
                btn.setBackground(new Color(255, 140, 0)); 
                btn.setText("🔥 LET'S GO!");
            }
            public void mouseExited(MouseEvent e) { 
                btn.setBackground(Color.BLACK); 
                btn.setText("🚀 ENTER THE HUB");
            }
        });

        gbc.gridy = 0; login.add(title, gbc);
        gbc.gridy = 1; login.add(subtitle, gbc);
        gbc.gridy = 2; gbc.insets = new Insets(20,0,10,0); login.add(userField, gbc);
        gbc.gridy = 3; gbc.insets = new Insets(10,0,0,0); login.add(btn, gbc);

        btn.addActionListener(e -> {
            username = userField.getText().trim();
            if (!username.isEmpty()) {
                loadXP();
                showMenu();
            }
        });

        mainPanel.add(login, "login");
        card.show(mainPanel, "login");
    }

    // ================= MENU =================
    void showMenu() {

        JPanel menu = new JPanel() {
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                Graphics2D g2 = (Graphics2D) g;
                GradientPaint gp = new GradientPaint(0, 0, new Color(0, 100, 255), getWidth(), getHeight(), new Color(200, 0, 255));
                g2.setPaint(gp);
                g2.fillRect(0, 0, getWidth(), getHeight());
            }
        };

        menu.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(15, 10, 15, 10);

        JLabel label = new JLabel("👾 Welcome " + username + " | XP: " + xp + " ✨");
        label.setFont(new Font("Arial", Font.BOLD, 24));
        label.setForeground(Color.WHITE);

        JButton dino = createGameButton("🦖 Dino Dash", "Run & Jump!");
        JButton candy = createGameButton("🍭 Candy Swap", "Match the Madness!");
        JButton ttt = createGameButton("❌⭕ Tic Tac Toe", "Classic Battle");

        gbc.gridy=0; menu.add(label, gbc);
        gbc.gridy=1; menu.add(dino, gbc);
        gbc.gridy=2; menu.add(candy, gbc);
        gbc.gridy=3; menu.add(ttt, gbc);

        dino.addActionListener(e -> startDino());
        candy.addActionListener(e -> startCandy());
        ttt.addActionListener(e -> startTTT());

        mainPanel.add(menu, "menu");
        card.show(mainPanel, "menu");
    }

    JButton createGameButton(String text, String subtitle) {
        JButton btn = new JButton("<html><center>" + text + "<br><font size='3' color='#AAAAAA'>" + subtitle + "</font></center></html>");
        btn.setFont(new Font("Arial", Font.BOLD, 18));
        btn.setBackground(new Color(30, 30, 30));
        btn.setForeground(Color.WHITE);
        btn.setPreferredSize(new Dimension(260, 70));
        btn.setFocusPainted(false);

        btn.addMouseListener(new MouseAdapter() {
            public void mouseEntered(MouseEvent e) { 
                btn.setBackground(new Color(255, 165, 0)); 
            }
            public void mouseExited(MouseEvent e) { 
                btn.setBackground(new Color(30, 30, 30)); 
            }
        });
        return btn;
    }

    // ================= MONGO =================
    void loadXP() {
        Document doc = collection.find(new Document("username", username)).first();
        if (doc != null) xp = doc.getInteger("xp", 0);
    }

    void saveXP(String game, int score) {
        xp += score;

        Document update = new Document("username", username)
                .append("xp", xp)
                .append("lastGame", game)
                .append("lastPlayed", LocalDate.now().toString());

        collection.updateOne(
                new Document("username", username),
                new Document("$set", update),
                new UpdateOptions().upsert(true)
        );
    }

    // ================= DINO =================
    void startDino() {

        JPanel panel = new JPanel() {

            int playerX = 100, playerY = 300;
            int velocity = 0;
            boolean jumping = false;
            boolean gameOver = false;

            int groundOffset = 0;

            Rectangle obstacle = new Rectangle(700, 300, 20, 40);

            int score = 0;
            long startTime = System.currentTimeMillis();

            Timer timer;

            {
                setFocusable(true);

                addKeyListener(new KeyAdapter() {
                    public void keyPressed(KeyEvent e) {

                        if (e.getKeyCode() == KeyEvent.VK_SPACE && !jumping) {
                            velocity = -16;   // slightly higher jump for fun
                            jumping = true;
                        }

                        if (gameOver && e.getKeyCode() == KeyEvent.VK_R) {
                            restart();
                        }
                    }
                });

                timer = new Timer(20, e -> updateGame());
                timer.start();
            }

            void updateGame() {

                if (!gameOver) {

                    obstacle.x -= 7;   // slightly faster for more challenge

                    if (obstacle.x < -20) {
                        obstacle.x = 800;
                        // Creative touch: random obstacle height sometimes
                        if (Math.random() > 0.7) obstacle.height = 50;
                        else obstacle.height = 40;
                    }

                    groundOffset -= 6;

                    velocity += 1;
                    playerY += velocity;

                    if (playerY >= 300) {
                        playerY = 300;
                        velocity = 0;
                        jumping = false;
                    }

                    Rectangle player = new Rectangle(playerX, playerY, 20, 35);

                    if (player.intersects(obstacle)) {
                        gameOver = true;
                        timer.stop();
                        saveXP("Dino", score);
                    }

                    score = (int)((System.currentTimeMillis() - startTime) / 800); // slower score increase for balance
                }

                repaint();
            }

            void restart() {
                playerY = 300;
                velocity = 0;
                gameOver = false;
                startTime = System.currentTimeMillis();
                obstacle.x = 800;
                obstacle.height = 40;
                timer.start();
            }

            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                Graphics2D g2 = (Graphics2D) g;

                // SKY with better gradient
                GradientPaint gp = new GradientPaint(0, 0,
                        new Color(135, 206, 250),
                        0, getHeight(),
                        new Color(70, 130, 255));
                g2.setPaint(gp);
                g2.fillRect(0, 0, getWidth(), getHeight());

                // SUN (creative touch)
                g2.setColor(new Color(255, 220, 0));
                g2.fillOval(600, 60, 60, 60);
                g2.setColor(new Color(255, 240, 100));
                g2.fillOval(610, 70, 40, 40);

                // RAINBOW (more vibrant)
                Color[] rainbow = {Color.RED, Color.ORANGE, Color.YELLOW, Color.GREEN, Color.CYAN, new Color(75,0,130), new Color(148,0,211)};
                for (int i = 0; i < rainbow.length; i++) {
                    g2.setColor(rainbow[i]);
                    g2.setStroke(new BasicStroke(8));
                    g2.drawArc((groundOffset / 2) % 900 - 350, 30 + i * 8, 700, 280, 0, 180);
                }

                // CLOUDS with better movement
                g2.setColor(new Color(255,255,255,220));
                int cloudX = (-groundOffset * 2) % 900;
                g2.fillOval(cloudX, 70, 90, 35);
                g2.fillOval(cloudX + 30, 55, 70, 40);
                g2.fillOval(cloudX + 300, 110, 85, 38);

                // GROUND with grass texture feel
                g2.setColor(new Color(34, 139, 34));
                g2.fillRect(0, 330, 800, 80);
                g2.setColor(new Color(50, 205, 50));
                g2.fillRect(0, 325, 800, 10);

                // STICKMAN (more lively)
                g2.setColor(Color.BLACK);
                g2.setStroke(new BasicStroke(3.5f));

                // head
                g2.drawOval(playerX + 4, playerY - 2, 13, 13);
                // eye (fun touch when jumping)
                g2.setColor(jumping ? Color.RED : Color.WHITE);
                g2.fillOval(playerX + 9, playerY + 3, 4, 4);

                // body
                g2.setColor(Color.BLACK);
                g2.drawLine(playerX + 10, playerY + 11, playerX + 10, playerY + 28);

                // arms (more dynamic)
                int armSwing = jumping ? 8 : (score % 3 == 0 ? 5 : -3);
                g2.drawLine(playerX + 10, playerY + 16, playerX - 3, playerY + 18 + armSwing);
                g2.drawLine(playerX + 10, playerY + 16, playerX + 23, playerY + 19 - armSwing);

                // legs animation (improved)
                if (jumping) {
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 3, playerY + 40);
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 17, playerY + 40);
                } else if (score % 2 == 0) {
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 2, playerY + 38);
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 18, playerY + 38);
                } else {
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 6, playerY + 39);
                    g2.drawLine(playerX + 10, playerY + 28, playerX + 14, playerY + 39);
                }

                // OBSTACLE - made it look like a rock/cactus
                g2.setColor(new Color(220, 20, 60));
                g2.fillRect(obstacle.x, obstacle.y, 22, obstacle.height);
                g2.setColor(Color.DARK_GRAY);
                g2.fillRect(obstacle.x + 5, obstacle.y + 5, 12, 8);

                // SCORE with shadow effect
                g2.setColor(new Color(0,0,0,80));
                g2.drawString("Score: " + score, 22, 22);
                g2.setColor(Color.WHITE);
                g2.drawString("Score: " + score, 20, 20);

                if (gameOver) {
                    g2.setColor(new Color(255, 0, 0, 220));
                    g2.setFont(new Font("Arial", Font.BOLD, 28));
                    g2.drawString("💥 GAME OVER 💥", 250, 180);
                    g2.setFont(new Font("Arial", Font.PLAIN, 18));
                    g2.drawString("Press R to Restart", 310, 220);
                }
            }
        };

        mainPanel.add(panel, "dino");
        card.show(mainPanel, "dino");
        panel.requestFocusInWindow();
    }

    // ================= TIC TAC TOE ================= (small polish only)
    void startTTT() {
        // Same as before but with better styling
        JPanel panel = new JPanel(new GridLayout(3,3, 8, 8));
        panel.setBackground(new Color(40,40,60));
        
        JButton[] btns = new JButton[9];
        char[] board = new char[9];
        final char[] current = {'X'};

        JLabel status = new JLabel("Player X's Turn", JLabel.CENTER);
        status.setFont(new Font("Arial",Font.BOLD,24));
        status.setForeground(Color.WHITE);

        JPanel wrapper = new JPanel(new BorderLayout());
        wrapper.setBackground(new Color(30,30,50));
        wrapper.add(status,BorderLayout.NORTH);
        wrapper.add(panel,BorderLayout.CENTER);

        for(int i=0;i<9;i++){
            int idx=i;
            btns[i]=new JButton("");
            btns[i].setFont(new Font("Arial",Font.BOLD,48));
            btns[i].setBackground(new Color(60,60,80));
            btns[i].setForeground(Color.WHITE);
            btns[i].setFocusPainted(false);
            panel.add(btns[i]);

            btns[i].addActionListener(e->{
                if(board[idx]==0){

                    board[idx]=current[0];
                    btns[idx].setText(""+current[0]);

                    if(current[0]=='X') btns[idx].setForeground(new Color(255, 80, 80));
                    else btns[idx].setForeground(new Color(80, 180, 255));

                    if(checkWin(board,current[0])){
                        JOptionPane.showMessageDialog(null,"🎉 Player "+current[0]+" WINS! 🎉");
                        saveXP("TicTacToe",15);   // slightly better reward
                        showMenu();
                        return;
                    }

                    if(isBoardFull(board)){
                        JOptionPane.showMessageDialog(null,"🤝 It's a Tie!");
                        showMenu();
                        return;
                    }

                    current[0]=(current[0]=='X')?'O':'X';
                    status.setText("Player "+current[0]+" Turn");
                }
            });
        }

        mainPanel.add(wrapper,"ttt");
        card.show(mainPanel,"ttt");
    }

    boolean checkWin(char[] b,char p){
        int[][] w={{0,1,2},{3,4,5},{6,7,8},{0,3,6},{1,4,7},{2,5,8},{0,4,8},{2,4,6}};
        for(int[] x:w)
            if(b[x[0]]==p && b[x[1]]==p && b[x[2]]==p) return true;
        return false;
    }

    boolean isBoardFull(char[] b){
        for(char c:b) if(c==0) return false;
        return true;
    }

    // ================= CANDY ================= (kept logic same, added polish)
    void startCandy() {

        JPanel panel = new JPanel(new BorderLayout());
        JPanel gridPanel = new JPanel(new GridLayout(5,5, 6, 6));
        gridPanel.setBackground(new Color(40,40,60));

        JButton[][] grid = new JButton[5][5];

        Color[] colors={
                new Color(255,105,180),  // hot pink
                new Color(255,165,0),    // orange
                new Color(0, 191, 255),  // deep sky blue
                new Color(138,43,226),   // blue violet
                new Color(50,205,50)     // lime green
        };

        JLabel scoreLabel=new JLabel("Score: 0",JLabel.CENTER);
        scoreLabel.setFont(new Font("Arial",Font.BOLD,24));
        scoreLabel.setForeground(Color.WHITE);

        int[] score={0};
        Random r=new Random();
        final int[] selected={-1,-1};

        panel.setBackground(new Color(30,30,50));
        panel.add(scoreLabel,BorderLayout.NORTH);
        panel.add(gridPanel,BorderLayout.CENTER);

        gridPanel.setFocusable(true);

        for(int i=0;i<5;i++){
            for(int j=0;j<5;j++){

                int row=i,col=j;
                grid[i][j]=new JButton();
                grid[i][j].setBackground(colors[r.nextInt(colors.length)]);
                grid[i][j].setFocusPainted(false);
                gridPanel.add(grid[i][j]);

                grid[i][j].addActionListener(e->{
                    if(selected[0] != -1){
                        grid[selected[0]][selected[1]].setBorder(null);
                    }
                    selected[0]=row;
                    selected[1]=col;
                    grid[row][col].setBorder(BorderFactory.createLineBorder(Color.YELLOW,4));
                    gridPanel.requestFocusInWindow();
                });
            }
        }

        gridPanel.addKeyListener(new KeyAdapter(){
            public void keyPressed(KeyEvent e){

                int i=selected[0],j=selected[1];
                if(i==-1) return;

                int ni=i,nj=j;
                if(e.getKeyCode()==KeyEvent.VK_UP) ni--;
                if(e.getKeyCode()==KeyEvent.VK_DOWN) ni++;
                if(e.getKeyCode()==KeyEvent.VK_LEFT) nj--;
                if(e.getKeyCode()==KeyEvent.VK_RIGHT) nj++;

                if(ni>=0 && ni<5 && nj>=0 && nj<5){

                    Color temp=grid[i][j].getBackground();
                    grid[i][j].setBackground(grid[ni][nj].getBackground());
                    grid[ni][nj].setBackground(temp);

                    grid[i][j].setBorder(null);
                    selected[0]=-1;

                    checkMatches(grid,score,scoreLabel);
                    checkWinCandy(grid,score,scoreLabel);
                }
            }
        });

        mainPanel.add(panel,"candy");
        card.show(mainPanel,"candy");
        gridPanel.requestFocusInWindow();   // better UX
    }

    void checkMatches(JButton[][] grid,int[] score,JLabel label){
        // same logic
        for(int i=0;i<5;i++)
            for(int j=0;j<3;j++){
                Color c=grid[i][j].getBackground();
                if(c.equals(grid[i][j+1].getBackground()) &&
                   c.equals(grid[i][j+2].getBackground())){
                    score[0]+=15;   // better points
                }
            }

        for(int j=0;j<5;j++)
            for(int i=0;i<3;i++){
                Color c=grid[i][j].getBackground();
                if(c.equals(grid[i+1][j].getBackground()) &&
                   c.equals(grid[i+2][j].getBackground())){
                    score[0]+=15;
                }
            }

        label.setText("Score: "+score[0]);
    }

    void checkWinCandy(JButton[][] grid,int[] score,JLabel label){
        Color first=grid[0][0].getBackground();

        for(int i=0;i<5;i++)
            for(int j=0;j<5;j++)
                if(!grid[i][j].getBackground().equals(first))
                    return;

        score[0]+=150;   // bigger reward
        label.setText("Score: "+score[0]);

        JOptionPane.showMessageDialog(null,"🌟 ALL CLEAR! YOU WON BIG! +150 XP");
        saveXP("Candy",score[0]);
        showMenu();
    }

    public static void main(String[] args){
        new PiggyGame();
    }
}
