# ⚔️ Spring Bootで作る「マルチプレイヤーじゃんけん（WebSocket）」

この教材では、Spring Bootを使って**リアルタイム通信によるマルチプレイヤーじゃんけん**を作ります。  
WebSocketを利用して、**複数の部屋（ルーム）**で独立して対戦できるようにします。

---

## 🎯 学習目標

- WebSocket通信の基本を理解する  
- Spring Bootでリアルタイムサーバーを構築する  
- 部屋ごとの接続管理とメッセージ送受信を実装する

---

## 🧱 プロジェクト構成

```
multiplayer-janken/
├─ src/main/java/com/example/janken/
│   ├─ JankenApplication.java
│   ├─ config/WebSocketConfig.java
│   ├─ controller/JankenController.java
│   ├─ model/Message.java
│   ├─ model/Room.java
│   └─ service/RoomService.java
└─ src/main/resources/static/
    └─ index.html
```

---

## ⚙️ 1. 環境設定

`build.gradle` に以下を追加します。

```gradle
plugins {
    id 'org.springframework.boot' version '3.3.0'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'java'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## 🚀 2. アプリケーション起動クラス

```java
package com.example.janken;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class JankenApplication {
    public static void main(String[] args) {
        SpringApplication.run(JankenApplication.class, args);
    }
}
```

---

## 🛰️ 3. WebSocket設定

```java
package com.example.janken.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

import com.example.janken.controller.JankenController;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    private final JankenController jankenController;

    public WebSocketConfig(JankenController jankenController) {
        this.jankenController = jankenController;
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(jankenController, "/ws/janken").setAllowedOrigins("*");
    }
}
```

---

## 🎮 4. モデル定義

### Message.java
```java
package com.example.janken.model;

public class Message {
    private String type; // "join", "move", "result"
    private String user;
    private String room;
    private String hand; // "rock", "paper", "scissors"

    public Message() {}

    public Message(String type, String user, String room, String hand) {
        this.type = type;
        this.user = user;
        this.room = room;
        this.hand = hand;
    }

    public String getType() { return type; }
    public String getUser() { return user; }
    public String getRoom() { return room; }
    public String getHand() { return hand; }

    public void setType(String type) { this.type = type; }
    public void setUser(String user) { this.user = user; }
    public void setRoom(String room) { this.room = room; }
    public void setHand(String hand) { this.hand = hand; }
}
```

### Room.java
```java
package com.example.janken.model;

import org.springframework.web.socket.WebSocketSession;
import java.util.*;

public class Room {
    private String roomId;
    private Map<String, WebSocketSession> players = new HashMap<>();
    private Map<String, String> hands = new HashMap<>();

    public Room(String roomId) {
        this.roomId = roomId;
    }

    public void addPlayer(String user, WebSocketSession session) {
        players.put(user, session);
    }

    public void removePlayer(String user) {
        players.remove(user);
        hands.remove(user);
    }

    public void setHand(String user, String hand) {
        hands.put(user, hand);
    }

    public boolean isReady() {
        return hands.size() == 2;
    }

    public Map<String, WebSocketSession> getPlayers() {
        return players;
    }

    public Map<String, String> getHands() {
        return hands;
    }

    public String getRoomId() {
        return roomId;
    }
}
```

---

## 🧠 5. RoomService.java

```java
package com.example.janken.service;

import com.example.janken.model.Room;
import java.util.concurrent.ConcurrentHashMap;

import org.springframework.stereotype.Service;

@Service
public class RoomService {
    private final ConcurrentHashMap<String, Room> rooms = new ConcurrentHashMap<>();

    public Room getOrCreateRoom(String roomId) {
        return rooms.computeIfAbsent(roomId, Room::new);
    }

    public void removeRoom(String roomId) {
        rooms.remove(roomId);
    }
}
```

---

## 🕹️ 6. WebSocketコントローラ

```java
package com.example.janken.controller;

import com.example.janken.model.Message;
import com.example.janken.model.Room;
import com.example.janken.service.RoomService;
import com.fasterxml.jackson.databind.ObjectMapper;

import org.springframework.stereotype.Component;
import org.springframework.web.socket.*;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.IOException;

@Component
public class JankenController extends TextWebSocketHandler {

    private final RoomService roomService;
    private final ObjectMapper mapper = new ObjectMapper();

    public JankenController(RoomService roomService) {
        this.roomService = roomService;
    }

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) throws IOException {
        Message msg = mapper.readValue(message.getPayload(), Message.class);
        Room room = roomService.getOrCreateRoom(msg.getRoom());

        switch (msg.getType()) {
            case "join" -> {
                room.addPlayer(msg.getUser(), session);
                broadcast(room, msg.getUser() + " が参加しました。");
            }
            case "move" -> {
                room.setHand(msg.getUser(), msg.getHand());
                if (room.isReady()) {
                    String result = judge(room);
                    broadcast(room, result);
                    room.getHands().clear();
                }
            }
        }
    }

    private String judge(Room room) {
        var hands = room.getHands();
        var players = hands.keySet().toArray(new String[0]);
        String p1 = players[0], p2 = players[1];
        String h1 = hands.get(p1), h2 = hands.get(p2);

        if (h1.equals(h2)) return "引き分け！ (" + h1 + ")";
        if ((h1.equals("rock") && h2.equals("scissors")) ||
            (h1.equals("scissors") && h2.equals("paper")) ||
            (h1.equals("paper") && h2.equals("rock"))) {
            return p1 + " の勝ち！ (" + h1 + " vs " + h2 + ")";
        } else {
            return p2 + " の勝ち！ (" + h1 + " vs " + h2 + ")";
        }
    }

    private void broadcast(Room room, String text) throws IOException {
        for (WebSocketSession s : room.getPlayers().values()) {
            if (s.isOpen()) s.sendMessage(new TextMessage(text));
        }
    }
}
```

---

## 🧩 7. クライアント側HTML

`src/main/resources/static/index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>マルチプレイヤーじゃんけん</title>
</head>
<body>
  <h1>⚔️ マルチプレイヤーじゃんけん</h1>
  <input id="room" placeholder="部屋番号">
  <input id="user" placeholder="ユーザー名">
  <button onclick="joinRoom()">参加</button>

  <div id="controls" style="display:none;">
    <button onclick="sendMove('rock')">✊ グー</button>
    <button onclick="sendMove('scissors')">✌ チョキ</button>
    <button onclick="sendMove('paper')">🖐 パー</button>
  </div>

  <pre id="log"></pre>

  <script>
    let ws, user, room;

    function joinRoom() {
      user = document.getElementById("user").value;
      room = document.getElementById("room").value;
      ws = new WebSocket("ws://localhost:8080/ws/janken");

      ws.onopen = () => {
        ws.send(JSON.stringify({type: "join", user, room}));
        document.getElementById("controls").style.display = "block";
      };

      ws.onmessage = (e) => {
        document.getElementById("log").textContent += e.data + "\n";
      };
    }

    function sendMove(hand) {
      ws.send(JSON.stringify({type: "move", user, room, hand}));
    }
  </script>
</body>
</html>
```

---

## 🧪 8. 実行とテスト

1. Spring Bootを起動します：
   ```bash
   ./gradlew bootRun
   ```
2. ブラウザで以下にアクセス：
   ```
   http://localhost:8080
   ```
3. 2つのブラウザを開いて同じ部屋番号を入力し、じゃんけんしてみましょう。

---

## 🧭 発展課題

- 部屋ごとにプレイヤー数制限を設ける  
- 勝敗履歴を保存してランキング化  
- WebSocket → STOMP に拡張してチャット機能を追加  

---

## ✅ まとめ

この教材で学んだこと：
- Spring BootによるWebSocket実装方法  
- ルーム管理とプレイヤー同期の考え方  
- JSONメッセージのシリアライズとハンドリング

---
