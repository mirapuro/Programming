# 🎮 Java 初学者向け教材：コンソールで遊ぶシンプルなオセロ

この教材では、Java の基本文法を学びながら「オセロ（リバーシ）」をコンソールで実装する方法を学びます。すべてのコードに丁寧な日本語コメントを記載しており、Java 初学者でも安心して学べる構成です。

---

## 📚 学べること

- Java の基本構文（配列、条件分岐、ループ、クラス）
- ゲームロジックの設計（オセロのルールに基づく実装）
- メソッドの分割と再利用
- コンソールアプリケーションの作成

---

## 📁 プロジェクト構成

- OthelloGame.java // ソースコード
- README.md // この教材

---

## 🚀 実行方法

Java がインストールされている環境で以下を実行します：

```bash
javac OthelloGame.java
java OthelloGame
```
---

## 🧩 ソースコード（OthelloGame.java）
以下はコンソールで動作するシンプルなオセロの完全なコードです。各行に初心者向けのコメントを記載しています。

```java
import java.util.Scanner; // ユーザー入力を受け取るため

public class OthelloGame {
    // 定数定義
    static final int SIZE = 8; // 盤面のサイズ（8x8）
    static final char EMPTY = '.'; // 空白マス
    static final char BLACK = 'B'; // 黒石
    static final char WHITE = 'W'; // 白石

    // 盤面を表す2次元配列
    static char[][] board = new char[SIZE][SIZE];

    // 現在のプレイヤー（黒からスタート）
    static char currentPlayer = BLACK;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        initializeBoard();  // 初期盤面の設定
        printBoard();       // 初期盤面の表示

        while (true) {
            System.out.println(currentPlayer + " の番です。行と列をスペースで入力してください（例: 2 3）：");

            int row = scanner.nextInt();   // 行
            int col = scanner.nextInt();   // 列

            if (isValidMove(row, col, currentPlayer)) {
                makeMove(row, col, currentPlayer); // 石を置いて裏返す
                switchPlayer();                    // プレイヤー交代
                printBoard();                      // 盤面表示
            } else {
                System.out.println("無効な手です。もう一度入力してください。");
            }
        }
    }

    // 初期盤面の配置（中央4つの石）
    static void initializeBoard() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                board[i][j] = EMPTY;
            }
        }
        board[3][3] = WHITE;
        board[3][4] = BLACK;
        board[4][3] = BLACK;
        board[4][4] = WHITE;
    }

    // 盤面を表示
    static void printBoard() {
        System.out.print("  ");
        for (int i = 0; i < SIZE; i++) {
            System.out.print(i + " ");
        }
        System.out.println();

        for (int i = 0; i < SIZE; i++) {
            System.out.print(i + " ");
            for (int j = 0; j < SIZE; j++) {
                System.out.print(board[i][j] + " ");
            }
            System.out.println();
        }
    }

    // 指定したマスが有効な手かを判定
    static boolean isValidMove(int row, int col, char player) {
        if (!isInBounds(row, col) || board[row][col] != EMPTY) {
            return false;
        }

        char opponent = (player == BLACK) ? WHITE : BLACK;

        int[] dx = {-1, -1, -1, 0, 1, 1, 1, 0};
        int[] dy = {-1, 0, 1, 1, 1, 0, -1, -1};

        for (int d = 0; d < 8; d++) {
            int x = row + dx[d];
            int y = col + dy[d];

            boolean hasOpponentBetween = false;

            while (isInBounds(x, y) && board[x][y] == opponent) {
                x += dx[d];
                y += dy[d];
                hasOpponentBetween = true;
            }

            if (hasOpponentBetween && isInBounds(x, y) && board[x][y] == player) {
                return true;
            }
        }

        return false;
    }

    // 石を置いて、間の相手石を裏返す
    static void makeMove(int row, int col, char player) {
        board[row][col] = player;

        char opponent = (player == BLACK) ? WHITE : BLACK;

        int[] dx = {-1, -1, -1, 0, 1, 1, 1, 0};
        int[] dy = {-1, 0, 1, 1, 1, 0, -1, -1};

        for (int d = 0; d < 8; d++) {
            int x = row + dx[d];
            int y = col + dy[d];

            StringBuilder toFlip = new StringBuilder();

            while (isInBounds(x, y) && board[x][y] == opponent) {
                toFlip.append(x).append(",").append(y).append(";");
                x += dx[d];
                y += dy[d];
            }

            if (isInBounds(x, y) && board[x][y] == player) {
                String[] flips = toFlip.toString().split(";");
                for (String f : flips) {
                    if (!f.isEmpty()) {
                        String[] pos = f.split(",");
                        int fx = Integer.parseInt(pos[0]);
                        int fy = Integer.parseInt(pos[1]);
                        board[fx][fy] = player;
                    }
                }
            }
        }
    }

    // プレイヤーの交代
    static void switchPlayer() {
        currentPlayer = (currentPlayer == BLACK) ? WHITE : BLACK;
    }

    // 範囲内かどうかの判定
    static boolean isInBounds(int x, int y) {
        return x >= 0 && x < SIZE && y >= 0 && y < SIZE;
    }
}
```

---

## 💡 補足・学習ポイント

| 概念     | 内容                              |
| ------ | ------------------------------- |
| 配列     | 2次元配列で盤面を管理                     |
| 条件分岐   | if文で入力チェック・合法手かどうか判定            |
| ループ    | for文や while文で方向ごとの探索と裏返しを実装     |
| 入力処理   | Scanner を使ってユーザーからの手入力を受け取る     |
| メソッド分割 | 処理ごとにメソッドを分け、見やすく・再利用可能な構成にしている |

---

## 🧠 発展課題

- パス処理（置ける手がなければスキップ）

- 勝敗判定（石の数で決着）

- 手番が無くなったらゲーム終了

- 簡単なAIを追加（ランダム or 最適手探索）

- GUI対応（Swing / JavaFX）

---

## 📝 ライセンス

MIT License（学習・教育目的で自由に使用・改変可能）

## 🙌 貢献

バグ報告、機能追加の提案、質問などあればお気軽に Pull Request や Issue を作成してください。
