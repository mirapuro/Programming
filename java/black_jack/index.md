# 🃏 Java ブラックジャック（Blackjack）入門教材

この教材では、Javaを使ってブラックジャックのゲームを段階的に作っていきます。  
各ステップごとにコードと解説があり、学習者が理解しながら実装を進められます。

---

## 目次

- [STEP 1: Cardクラスを作ろう](#step-1-cardクラスを作ろう)  
- [STEP 2: Deckクラスを実装しよう](#step-2-deckクラスを実装しよう)  
- [STEP 3: PlayerHandクラスで手札を管理しよう](#step-3-playerhandクラスで手札を管理しよう)  
- [STEP 4: HitとStandのロジックを作ろう](#step-4-hitとstandのロジックを作ろう)  
- [STEP 5: 勝敗判定・バースト処理・ディーラーの動きを追加しよう](#step-5-勝敗判定バースト処理ディーラーの動きを追加しよう)  
- [STEP 6: ゲームのメインループを完成させよう](#step-6-ゲームのメインループを完成させよう)

---

## STEP 1: Cardクラスを作ろう

カード1枚を表すクラスです。  
スート（♠︎, ♥︎, ♦︎, ♣︎）と数字（1〜13）を持ち、ブラックジャックの点数計算のために値を取得するメソッドも追加します。

```java
// Card.java
public class Card {
    private String suit;   // スート（♠︎, ♥︎, ♦︎, ♣︎）
    private int number;    // カードの数字（1=エース, 11=ジャック, 12=クイーン, 13=キング）

    public Card(String suit, int number) {
        this.suit = suit;
        this.number = number;
    }

    public String getSuit() {
        return suit;
    }

    public int getNumber() {
        return number;
    }

    // ブラックジャックでのカードの点数を返す（Aは11点として扱うが、後で調整）
    public int getValue() {
        if (number >= 10) { // 10, J, Q, K は10点
            return 10;
        } else if (number == 1) { // エースは最初11点として扱う
            return 11;
        } else {
            return number;
        }
    }

    @Override
    public String toString() {
        String numStr;
        switch(number) {
            case 1: numStr = "A"; break;
            case 11: numStr = "J"; break;
            case 12: numStr = "Q"; break;
            case 13: numStr = "K"; break;
            default: numStr = String.valueOf(number);
        }
        return suit + numStr;
    }
}
```
### ポイント
- カードのスートと数字を保持する
- 点数の取得ロジックを入れている
- toString()で「♠A」など見やすく表示できる

---

## STEP 2: Deckクラスを実装しよう
52枚のカードの山札を作成し、シャッフル・1枚ずつ配る機能を実装します。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

// Deck.java
public class Deck {
    private List<Card> cards;

    public Deck() {
        cards = new ArrayList<>();
        String[] suits = {"♠", "♥", "♦", "♣"};
        // 1〜13のカードを全スートで作る
        for (String suit : suits) {
            for (int num = 1; num <= 13; num++) {
                cards.add(new Card(suit, num));
            }
        }
    }

    // 山札をシャッフルする
    public void shuffle() {
        Collections.shuffle(cards);
    }

    // 山札から1枚カードを引く（取り除く）
    public Card draw() {
        if (cards.isEmpty()) {
            return null; // 山札が空ならnullを返す
        }
        return cards.remove(0);
    }

    // 残り枚数を返す
    public int remainingCards() {
        return cards.size();
    }
}
```

### ポイント
- ArrayList でカードを管理
- Collections.shuffle() で山札をシャッフル
- draw() でカードを1枚取り出す

---

## STEP 3: PlayerHandクラスで手札を管理しよう
プレイヤーやディーラーの持つ手札を管理し、合計点数を計算するクラスを作ります。

```java
import java.util.ArrayList;
import java.util.List;

// PlayerHand.java
public class PlayerHand {
    private List<Card> hand;

    public PlayerHand() {
        hand = new ArrayList<>();
    }

    // カードを追加
    public void addCard(Card card) {
        hand.add(card);
    }

    // 手札のカード一覧を取得
    public List<Card> getCards() {
        return hand;
    }

    // 合計点数を計算（エースは1または11として計算）
    public int getTotalValue() {
        int total = 0;
        int aceCount = 0;

        // まず全カードの点数合計（エースは11点としてカウント）
        for (Card card : hand) {
            total += card.getValue();
            if (card.getNumber() == 1) {
                aceCount++;
            }
        }

        // 合計が21を超える場合は、エースを11点から1点に変更する
        while (total > 21 && aceCount > 0) {
            total -= 10; // 11点を1点に変換した分を減算
            aceCount--;
        }

        return total;
    }

    // 手札を文字列にして表示
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Card card : hand) {
            sb.append(card.toString()).append(" ");
        }
        return sb.toString().trim();
    }
}
```

### ポイント
- 手札をListで管理
- エースの柔軟な点数計算（1か11）
- 手札を文字列で簡単に表示可能

---

## STEP 4: HitとStandのロジックを作ろう
プレイヤーの「ヒット（カードを引く）」と「スタンド（カードを引かずに待つ）」の操作を処理します。
今回はコンソール入力を想定し、Scannerで入力を受け取ります。

```java
import java.util.Scanner;

// BlackjackGame.java（メインクラスの一部）
public class BlackjackGame {
    private Deck deck;
    private PlayerHand playerHand;
    private PlayerHand dealerHand;
    private Scanner scanner;

    public BlackjackGame() {
        deck = new Deck();
        deck.shuffle();

        playerHand = new PlayerHand();
        dealerHand = new PlayerHand();

        scanner = new Scanner(System.in);
    }

    public void start() {
        // 最初に2枚ずつカードを配る
        playerHand.addCard(deck.draw());
        playerHand.addCard(deck.draw());

        dealerHand.addCard(deck.draw());
        dealerHand.addCard(deck.draw());

        System.out.println("ブラックジャックを開始します！");
        System.out.println("あなたの手札: " + playerHand);
        System.out.println("合計点数: " + playerHand.getTotalValue());

        // プレイヤーのターン
        while (true) {
            System.out.print("カードを引きますか？(hit/stand): ");
            String input = scanner.nextLine().trim().toLowerCase();

            if (input.equals("hit")) {
                Card newCard = deck.draw();
                System.out.println("引いたカード: " + newCard);
                playerHand.addCard(newCard);
                System.out.println("あなたの手札: " + playerHand);
                System.out.println("合計点数: " + playerHand.getTotalValue());

                if (playerHand.getTotalValue() > 21) {
                    System.out.println("バーストしました！あなたの負けです。");
                    return;
                }
            } else if (input.equals("stand")) {
                System.out.println("あなたはスタンドしました。");
                break;
            } else {
                System.out.println("入力が不正です。hit または stand を入力してください。");
            }
        }

        // 次のステップでディーラーの処理や勝敗判定を実装します。
    }

    public static void main(String[] args) {
        BlackjackGame game = new BlackjackGame();
        game.start();
    }
}
```

### ポイント
- Scanner でユーザー入力を取得
- hitでカードを引き、合計点数を更新
- バースト（21超え）判定で即敗北
- standでプレイヤーのターン終了

---

## STEP 5: 勝敗判定・バースト処理・ディーラーの動きを追加しよう

プレイヤーが「stand」した後、ディーラーは17点以上になるまでカードを引きます。  
その後、点数を比較して勝敗を判定します。

👩‍⚖️ ルール（一般的な簡易ルール）

- ディーラーは 17 点以上になるまでカードを引き続ける
- プレイヤーが21を超えていたら自動的に負け
- ディーラーが21を超えたらプレイヤーの勝ち
- それ以外は点数の高い方が勝ち。同点は引き分け

🎲 実装（BlackjackGame.java の start() の後半）

```java
// ディーラーのターン
System.out.println("ディーラーの手札: " + dealerHand);
System.out.println("合計点数: " + dealerHand.getTotalValue());

while (dealerHand.getTotalValue() < 17) {
    Card newCard = deck.draw();
    System.out.println("ディーラーがカードを引きました: " + newCard);
    dealerHand.addCard(newCard);
    System.out.println("ディーラーの手札: " + dealerHand);
    System.out.println("合計点数: " + dealerHand.getTotalValue());
}

// 勝敗判定
int playerScore = playerHand.getTotalValue();
int dealerScore = dealerHand.getTotalValue();

System.out.println("=== 結果発表 ===");
System.out.println("あなたの手札: " + playerHand + "（合計: " + playerScore + "）");
System.out.println("ディーラーの手札: " + dealerHand + "（合計: " + dealerScore + "）");

if (dealerScore > 21) {
    System.out.println("ディーラーがバースト！あなたの勝ちです！");
} else if (playerScore > dealerScore) {
    System.out.println("あなたの勝ちです！");
} else if (playerScore < dealerScore) {
    System.out.println("あなたの負けです！");
} else {
    System.out.println("引き分けです！");
}
```

### ポイント
- ディーラーのロジックは「17点未満ならヒット」
- プレイヤーがバーストしていなければディーラーと比較
- スコアの比較で勝敗を明示的に表示

---

## STEP 6: ゲームのメインループを完成させよう
最後に、1ゲーム終わった後に「もう一度プレイしますか？」と確認してループさせます。

🎯 ゲーム全体のループ

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    while (true) {
        BlackjackGame game = new BlackjackGame();
        game.start();

        System.out.print("もう一度プレイしますか？(yes/no): ");
        String input = scanner.nextLine().trim().toLowerCase();
        if (!input.equals("yes")) {
            System.out.println("ゲームを終了します。ありがとうございました！");
            break;
        }
        System.out.println("\n=========================\n");
    }
    scanner.close();
}
```

### 📌 これで完成！

- カード、デッキ、プレイヤー手札、ゲームロジックがすべて連携
- 段階的に作ることで、各構造の役割を理解しやすくなる

---

## 🎓 発展課題（上級者向け）
さらにスキルアップしたい方のために、以下の拡張を提案します：
| 機能            | 内容例                              |
| ------------- | -------------------------------- |
| GUI化（Swing）   | プレイヤー操作をボタンで行えるようにする             |
| 複数プレイヤー対応     | 複数人での対戦プレイ                       |
| 賭け金の導入（チップ管理） | 所持金や賭け金による勝敗の管理                  |
| サウンド効果追加      | 勝敗・バースト時に効果音を鳴らす（Java Sound API） |
| ファイル保存（戦績や勝率） | ゲーム履歴やハイスコアを記録                   |
| ユニットテストの導入    | 各クラスのテストコードをJUnitで書いて品質を高める      |

---

### ✅ 完成したファイル構成（推奨）
/Blackjack/
├── Card.java
├── Deck.java
├── PlayerHand.java
└── BlackjackGame.java

コンパイル・実行例：
```bash
javac Card.java Deck.java PlayerHand.java BlackjackGame.java
java BlackjackGame
```

### 🏁 お疲れさまでした！

この教材を通じて、以下のような Java の基本技術が身についたはずです：

- クラス設計（Card, Deck, PlayerHand）
- オブジェクト指向（メソッドの分離と再利用）
- コレクション（List, ArrayList）
- 入力処理（Scanner）
- 条件分岐・ループ・比較
- ステップバイステップの開発力

今後もたくさんの課題に挑戦してJavaプログラミングを学習していきましょう！
