/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/Classes/Class.java to edit this template
 */
package labiringame;

import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JSlider;
import javax.swing.event.ChangeEvent;
import javax.swing.event.ChangeListener;

public class MazeGame extends JPanel implements KeyListener {
    private static final int CELL_SIZE = 20; // Ukuran satu sel dalam piksel
    private static final int MAZE_SIZE = 20; // Ukuran labirin (jumlah sel)
    private static final int FRAME_SIZE = MAZE_SIZE * CELL_SIZE; // Ukuran jendela (dalam piksel)
    
    private static final int RED_DROID = 1;
    private static final int GREEN_DROID = 2;
    
    private static final int[][] DIRECTIONS = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}}; // Atas, Kanan, Bawah, Kiri
    
    private int[][] maze; // Representasi labirin dalam matriks
    private int redDroidX, redDroidY; // Posisi droid merah
    private int greenDroidX, greenDroidY; // Posisi droid hijau
    private boolean isRunning; // Status game berjalan atau dijeda
    
    private JButton startButton, pauseButton, shuffleButton, shuffleRedButton, shuffleGreenButton;
    private JSlider viewDistanceSlider;
    private JLabel redDroidLabel, greenDroidLabel;
    
    public MazeGame() {
        setPreferredSize(new Dimension(FRAME_SIZE, FRAME_SIZE));
        setBackground(Color.WHITE);
        setFocusable(true);
        addKeyListener(this);
        
        
        maze = new int[MAZE_SIZE][MAZE_SIZE];
        redDroidX = 0;
        redDroidY = 0;
        greenDroidX = MAZE_SIZE - 1;
        greenDroidY = MAZE_SIZE - 1;
        isRunning = false;
        
        startButton = new JButton("Start");
        startButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                isRunning = true;
                startButton.setEnabled(false);
            }
        });
        
        pauseButton = new JButton("Pause");
        pauseButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                isRunning = false;
                startButton.setEnabled(true);
            }
        });
        pauseButton.setEnabled(false);
        
        shuffleButton = new JButton("Shuffle Map");
        shuffleButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                shuffleMap();
                startButton.setEnabled(true);
            }
        });
        
        shuffleRedButton = new JButton("Shuffle Red Droid");
        shuffleRedButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                shuffleRedDroid();
            }
        });
        
        shuffleGreenButton = new JButton("Shuffle Green Droid");
        shuffleGreenButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                shuffleGreenDroid();
            }
        });
        
        viewDistanceSlider = new JSlider(JSlider.HORIZONTAL, 1, MAZE_SIZE, MAZE_SIZE);
        viewDistanceSlider.setMajorTickSpacing(1);
        viewDistanceSlider.setPaintTicks(true);
        viewDistanceSlider.setPaintLabels(true);
        viewDistanceSlider.addChangeListener(new ChangeListener() {
            public void stateChanged(ChangeEvent e) {
                repaint();
            }
        });
        
        redDroidLabel = new JLabel("Red Droid Position: (" + redDroidX + ", " + redDroidY + ")");
        greenDroidLabel = new JLabel("Green Droid Position: (" + greenDroidX + ", " + greenDroidY + ")");
        
        JFrame frame = new JFrame("Maze Game");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());
        
        JPanel buttonPanel = new JPanel();
buttonPanel.add(startButton);
buttonPanel.add(pauseButton);
buttonPanel.add(shuffleButton);
buttonPanel.add(shuffleRedButton);
buttonPanel.add(shuffleGreenButton);

frame.add(buttonPanel, BorderLayout.NORTH);
frame.add(viewDistanceSlider, BorderLayout.CENTER);
frame.add(redDroidLabel, BorderLayout.WEST);
frame.add(greenDroidLabel, BorderLayout.EAST);

frame.add(this, BorderLayout.SOUTH); // Tambahkan panel permainan ke bagian bawah frame

frame.pack();
frame.setResizable(false);
frame.setLocationRelativeTo(null);
frame.setVisible(true);
    }
    
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        
        // Gambar labirin
        for (int i = 0; i < MAZE_SIZE; i++) {
            for (int j = 0; j < MAZE_SIZE; j++) {
                if (maze[i][j] == 1) {
                    g.setColor(Color.BLACK);
                    g.fillRect(j * CELL_SIZE, i * CELL_SIZE, CELL_SIZE, CELL_SIZE);
                }
            }
        }
        
        // Gambar droid merah
        g.setColor(Color.RED);
        g.fillOval(redDroidX * CELL_SIZE, redDroidY * CELL_SIZE, CELL_SIZE, CELL_SIZE);
        
        // Gambar droid hijau
        g.setColor(Color.GREEN);
        g.fillOval(greenDroidX * CELL_SIZE, greenDroidY * CELL_SIZE, CELL_SIZE, CELL_SIZE);
        
        // Tampilkan jarak pandangan droid menggunakan garis putus-putus
        if (isRunning) {
            g.setColor(Color.BLUE);
            g.drawLine(redDroidX * CELL_SIZE + CELL_SIZE / 2, redDroidY * CELL_SIZE + CELL_SIZE / 2,
                    greenDroidX * CELL_SIZE + CELL_SIZE / 2, greenDroidY * CELL_SIZE + CELL_SIZE / 2);
        }
    }
    
    public void keyTyped(KeyEvent e) {}
    
    public void keyPressed(KeyEvent e) {
        if (!isRunning)
            return;
        
        int key = e.getKeyCode();
        
        if (key == KeyEvent.VK_UP)
            moveDroid(RED_DROID, -1, 0);
        else if (key == KeyEvent.VK_DOWN)
            moveDroid(RED_DROID, 1, 0);
        else if (key == KeyEvent.VK_LEFT)
            moveDroid(RED_DROID, 0, -1);
        else if (key == KeyEvent.VK_RIGHT)
            moveDroid(RED_DROID, 0, 1);
    }
    
    public void keyReleased(KeyEvent e) {}
    
    private void moveDroid(int droid, int dx, int dy) {
        int newX = 0, newY = 0;
        
        if (droid == RED_DROID) {
            newX = redDroidX + dx;
            newY = redDroidY + dy;
        } else if (droid == GREEN_DROID) {
            newX = greenDroidX + dx;
            newY = greenDroidY + dy;
        }
        
        if (isValidMove(newX, newY)) {
            if (droid == RED_DROID) {
                redDroidX = newX;
                redDroidY = newY;
                redDroidLabel.setText("Red Droid Position: (" + redDroidX + ", " + redDroidY + ")");
            } else if (droid == GREEN_DROID) {
                greenDroidX = newX;
                greenDroidY = newY;
                greenDroidLabel.setText("Green Droid Position: (" + greenDroidX + ", " + greenDroidY + ")");
            }
            
            repaint();
        }
    }
    
    private boolean isValidMove(int x, int y) {
        return x >= 0 && x < MAZE_SIZE && y >= 0 && y < MAZE_SIZE && maze[x][y] != 1;
    }
    
    private void shuffleMap() {
        // Buat labirin kosong
        for (int i = 0; i < MAZE_SIZE; i++) {
            for (int j = 0; j < MAZE_SIZE; j++) {
                maze[i][j] = 0;
            }
        }
        
        // Buat jalur labirin menggunakan DFS
        dfs(0, 0);
        
        // Acak posisi droid merah
        shuffleRedDroid();
        
        // Acak posisi droid hijau
        shuffleGreenDroid();
        
        repaint();
    }
    
    private void dfs(int x, int y) {
        List<Integer> directions = new ArrayList<>();
        for (int i = 0; i < 4; i++) {
            directions.add(i);
        }
        Collections.shuffle(directions);
        
        for (int direction : directions) {
            int newX = x + DIRECTIONS[direction][0] * 2;
            int newY = y + DIRECTIONS[direction][1] * 2;
            
            if (newX >= 0 && newX < MAZE_SIZE && newY >= 0 && newY < MAZE_SIZE && maze[newX][newY] == 0) {
                maze[x + DIRECTIONS[direction][0]][y + DIRECTIONS[direction][1]] = 1;
                maze[newX][newY] = 1;
                
                dfs(newX, newY);
            }
        }
    }
    
    private void shuffleRedDroid() {
        List<int[]> emptyCells = new ArrayList<>();
        for (int i = 0; i < MAZE_SIZE; i++) {
            for (int j = 0; j < MAZE_SIZE; j++) {
                if (maze[i][j] == 0) {
                    emptyCells.add(new int[]{i, j});
                }
            }
        }
        
        Collections.shuffle(emptyCells);
        
        redDroidX = emptyCells.get(0)[0];
        redDroidY = emptyCells.get(0)[1];
        
        redDroidLabel.setText("Red Droid Position: (" + redDroidX + ", " + redDroidY + ")");
        
        repaint();
    }
    
    private void shuffleGreenDroid() {
        List<int[]> emptyCells = new ArrayList<>();
        for (int i = 0; i < MAZE_SIZE; i++) {
            for (int j = 0; j < MAZE_SIZE; j++) {
                if (maze[i][j] == 0 && (i != redDroidX || j != redDroidY)) {
                    emptyCells.add(new int[]{i, j});
                }
            }
        }
        
        Collections.shuffle(emptyCells);
        
        greenDroidX = emptyCells.get(0)[0];
        greenDroidY = emptyCells.get(0)[1];
        
        greenDroidLabel.setText("Green Droid Position: (" + greenDroidX + ", " + greenDroidY + ")");
        
        repaint();
    }
    
    private void setViewDistance(int distance) {
        int redDroidCenterX = redDroidX * CELL_SIZE + CELL_SIZE / 2;
        int redDroidCenterY = redDroidY * CELL_SIZE + CELL_SIZE / 2;
        int greenDroidCenterX = greenDroidX * CELL_SIZE + CELL_SIZE / 2;
        int greenDroidCenterY = greenDroidY * CELL_SIZE + CELL_SIZE / 2;
        
        int dx = greenDroidCenterX - redDroidCenterX;
        int dy = greenDroidCenterY - redDroidCenterY;
        
        double magnitude = Math.sqrt(dx * dx + dy * dy);
        
        if (magnitude > distance * CELL_SIZE) {
            int newX = (int) (redDroidCenterX + (dx / magnitude) * distance * CELL_SIZE);
            int newY = (int) (redDroidCenterY + (dy / magnitude) * distance * CELL_SIZE);
            
            int newDroidX = newX / CELL_SIZE;
            int newDroidY = newY / CELL_SIZE;
            
            if (isValidMove(newDroidX, newDroidY)) {
                redDroidX = newDroidX;
                redDroidY = newDroidY;
                
                redDroidLabel.setText("Red Droid Position: (" + redDroidX + ", " + redDroidY + ")");
                repaint();
            }
        }
    }
    
    public static void main(String[] args) {
        new MazeGame();
    }
}
  

