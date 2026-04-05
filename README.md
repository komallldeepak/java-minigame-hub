# 🎮 Piggy Game Hub

A colorful and engaging **Java Swing** mini-games platform where you can level up your skills and collect XP!

Built as a student project with clean GUI, smooth animations, and persistent progress using **MongoDB**.

![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Swing](https://img.shields.io/badge/Swing-GUI-blue?style=for-the-badge)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)

## ✨ Features

- **Beautiful Login Screen** with gradient and hover effects
- **Main Menu** showing current XP
- **4 Mini Games**:
  - 🦖 **Dino Dash** – Endless runner with jumping (like Chrome Dino)
  - 🍭 **Candy Swap** – Match-3 style puzzle with keyboard controls
  - ❌⭕ **Tic-Tac-Toe** – Classic two-player game
  - 🚇 **Subway Surfers** – Endless runner (integrated via SubwayRunner)
- **XP & Progress System** – Earn XP from every game and save it to MongoDB
- **Modern UI** – Gradients, animations, hover effects, and fun emojis
- **Data Persistence** – MongoDB stores username, XP, last game, and last played date

## 🎯 How to Play

1. Run the application → Enter any username
2. Choose a game from the menu
3. Play and earn XP!
4. Your progress is automatically saved

### Controls:
- **Dino Dash**: `SPACE` to jump, `R` to restart
- **Candy Swap**: Arrow keys to swap after selecting a candy
- **Tic-Tac-Toe**: Click on the grid
- **Subway Surfers**: (depends on your SubwayRunner implementation)

## 🛠️ Technologies Used

- Java (Swing + AWT)
- MongoDB Java Driver (for saving player data)
- CardLayout for smooth screen navigation
- Custom painting with `Graphics2D` for games

## 🚀 How to Run

### Prerequisites
- Java 8 or higher installed
- MongoDB running locally on `mongodb://localhost:27017`
- (Optional) Import the project in Eclipse / IntelliJ / VS Code

### Steps
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/Piggy-Game-Hub.git
