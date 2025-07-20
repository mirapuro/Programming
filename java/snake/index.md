# 🐍 Java 初心者向け教材：スネークゲーム（Snake Game）

この教材では、Java Swing を使って「スネークゲーム」を実装します。タイマー処理やキーボード操作など、ゲーム開発の基礎を楽しく学ぶことができます。

---

## 🧑‍🏫 対象

- Java 初心者〜中級者
- GUI アプリケーションの入門を学びたい方
- タイマーやイベント処理に興味がある方

---

## 🎯 学べること

| 概要                     | 内容                                                                 |
|--------------------------|----------------------------------------------------------------------|
| GUIの基礎                | Swing フレーム、パネル、グラフィックス                              |
| タイマーとゲームループ  | javax.swing.Timer による定期描画処理                               |
| キー入力処理            | KeyListener による方向制御                                          |
| リスト操作               | スネークの体の移動と成長処理                                        |
| ランダム                 | エサの出現場所のランダム生成                                        |
| 衝突判定                 | 壁または自身とぶつかるとゲーム終了                                  |

---

## 🚀 実行方法

Java（JDK 8 以上）がインストールされた環境で以下を実行してください。

1. ファイル名を SnakeGame.java として保存
2. 以下のコマンドでコンパイル・実行：

```bash
javac SnakeGame.java
java SnakeGame
```
---

## 💻 ソースコード（SnakeGame.java）
```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.Random;

// メインクラス（フレーム作成）
public class SnakeGame extends JFrame {
    public SnakeGame() {
        setTitle("🐍 スネークゲーム - Java教材");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setResizable(false);
        add(new GamePanel()); // ゲームパネルを追加
        pack(); // パネルのサイズにウィンドウを合わせる
        setLocationRelativeTo(null); // 中央表示
        setVisible(true); // ウィンドウ表示
    }

    public static void main(String[] args) {
        new SnakeGame(); // ゲーム開始
    }
}

// ゲーム画面を描画するパネルクラス
class GamePanel extends JPanel implements ActionListener {
    // 定数（セルサイズ、画面サイズ）
    static final int CELL_SIZE = 25;
    static final int WIDTH = 600;
    static final int HEIGHT = 600;
    static final int TOTAL_CELLS = (WIDTH * HEIGHT) / (CELL_SIZE * CELL_SIZE);
    static final int DELAY = 100; // ミリ秒：ゲームの速度

    // スネークの位置（X, Y座標）
    ArrayList<Point> snake = new ArrayList<>();

    // 食べ物の位置
    Point food;

    // 方向（初期は右向き）
    char direction = 'R';

    // ゲーム状態
    boolean running = false;

    // タイマー（ゲームループ）
    Timer timer;

    // ランダムインスタンス
    Random random;

    // コンストラクタ
    public GamePanel() {
        setPreferredSize(new Dimension(WIDTH, HEIGHT));
        setBackground(Color.black);
        setFocusable(true);
        addKeyListener(new MyKeyAdapter());
        startGame(); // ゲーム開始
    }

    public void startGame() {
        random = new Random();
        snake.clear();
        // 初期スネークは3マス
        snake.add(new Point(5, 5));
        snake.add(new Point(4, 5));
        snake.add(new Point(3, 5));
        spawnFood(); // エサを出す
        running = true;
        timer = new Timer(DELAY, this); // ゲームループタイマー
        timer.start();
    }

    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        draw(g); // 描画処理
    }

    public void draw(Graphics g) {
        if (running) {
            // 食べ物の描画（赤い四角）
            g.setColor(Color.red);
            g.fillOval(food.x * CELL_SIZE, food.y * CELL_SIZE, CELL_SIZE, CELL_SIZE);

            // スネークの描画
            for (int i = 0; i < snake.size(); i++) {
                if (i == 0) {
                    g.setColor(Color.green); // 頭は緑
                } else {
                    g.setColor(new Color(0, 100, 0)); // 体は濃い緑
                }
                Point p = snake.get(i);
                g.fillRect(p.x * CELL_SIZE, p.y * CELL_SIZE, CELL_SIZE, CELL_SIZE);
            }

            // グリッド（オプション）
            /*
            g.setColor(Color.darkGray);
            for (int i = 0; i < HEIGHT / CELL_SIZE; i++) {
                g.drawLine(0, i * CELL_SIZE, WIDTH, i * CELL_SIZE);
                g.drawLine(i * CELL_SIZE, 0, i * CELL_SIZE, HEIGHT);
            }
            */
        } else {
            gameOver(g);
        }
    }

    public void move() {
        // 現在の頭の位置を取得
        Point head = new Point(snake.get(0));
        switch (direction) {
            case 'U': head.y -= 1; break;
            case 'D': head.y += 1; break;
            case 'L': head.x -= 1; break;
            case 'R': head.x += 1; break;
        }
        // 新しい頭を先頭に追加
        snake.add(0, head);

        // 食べ物に到達したか判定
        if (head.equals(food)) {
            spawnFood(); // 新しいエサを生成（長くなる）
        } else {
            snake.remove(snake.size() - 1); // 移動：最後尾を削除
        }
    }

    public void spawnFood() {
        while (true) {
            int x = random.nextInt(WIDTH / CELL_SIZE);
            int y = random.nextInt(HEIGHT / CELL_SIZE);
            Point newFood = new Point(x, y);
            if (!snake.contains(newFood)) {
                food = newFood;
                break;
            }
        }
    }

    public void checkCollisions() {
        Point head = snake.get(0);
        // 壁にぶつかったらゲームオーバー
        if (head.x < 0 || head.x >= WIDTH / CELL_SIZE || head.y < 0 || head.y >= HEIGHT / CELL_SIZE) {
            running = false;
        }
        // 自分自身にぶつかったらゲームオーバー
        for (int i = 1; i < snake.size(); i++) {
            if (head.equals(snake.get(i))) {
                running = false;
            }
        }
        if (!running) {
            timer.stop();
        }
    }

    public void gameOver(Graphics g) {
        g.setColor(Color.red);
        g.setFont(new Font("Arial", Font.BOLD, 48));
        FontMetrics fm = getFontMetrics(g.getFont());
        g.drawString("ゲームオーバー", (WIDTH - fm.stringWidth("ゲームオーバー")) / 2, HEIGHT / 2);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (running) {
            move();           // 移動
            checkCollisions(); // 衝突判定
        }
        repaint(); // 描画更新
    }

    // キー入力処理（方向変更）
    public class MyKeyAdapter extends KeyAdapter {
        @Override
        public void keyPressed(KeyEvent e) {
            switch (e.getKeyCode()) {
                case KeyEvent.VK_LEFT:
                    if (direction != 'R') direction = 'L';
                    break;
                case KeyEvent.VK_RIGHT:
                    if (direction != 'L') direction = 'R';
                    break;
                case KeyEvent.VK_UP:
                    if (direction != 'D') direction = 'U';
                    break;
                case KeyEvent.VK_DOWN:
                    if (direction != 'U') direction = 'D';
                    break;
            }
        }
    }
}
```
---
##🎓 発展課題（チャレンジ！）
- ゲームスコアの表示
- 難易度（速度）調整
- エサの種類（スコアに応じて色や効果を変える）
- ハイスコアの保存（ファイル入出力）
- サウンド追加（javax.sound.sampled)
