发布订阅模式顾名思义就是发布者和订阅者组成的设计模式，发布者发布消息，订阅了消息的订阅者收到通知，这里多了一个主体：消息，消息作为发布者和订阅者的中介，可以分发订阅和分发通知，举一个具体一点的例子：

> 王哥和刘哥都喜欢玩游戏，但是工作忙，并不想每天打开 steam 去搜游戏，希望 steam 在发布新游戏的时候通知他们。而且二位兴趣侧重有所不同，王哥喜欢 CRPG、JRPG、ACT，刘哥喜欢 RTS 和 MOBA，如果 steam 上线新游戏的时候一股脑推给二位，那手机就爆炸了，所以他们希望按照兴趣来收到推送。

以上是虚构的一个业务场景，我们整理一下信息

- 消息发布者 steam（Publisher）
- 消息订阅者王哥和刘哥（Subscriber）
- 需要一个中介来帮 steam 和用户之间决定哪些订阅和哪些推送（EventSystem/Observers）

---

首先来实现游戏平台 steam

```javascript
class GamePlatform {
  games = [];
  observers = [];

  constructor(locale = "zh-CN", name = "steam") {
    this.locale = locale;
    this.name = name;
  }

  // type TObserver = {
  //   id: string;
  //   name: string;
  //   onPublish: (game) => void;
  //   [others: string]: any;
  // }
  addObserver(observer) {
    if (!this.observers.find(({ id }) => id === observer.id)) {
      this.observers.push(observer);
    }
  }

  removeObserver(observerId) {
    this.observers = this.observers.filter(({ id }) => id !== observerId);
  }

  // type TGame = {
  //   id: string;
  //   name: string;
  //   type: string;
  //   [others: string]: any;
  // }
  publishGame(game) {
    if (this.games.find(({ id }) => id === game.id)) {
      this.games.push(game);
      this.observers.forEach((observer) => {
        observer.onPublish(game);
      });
    }
  }
}
```

---

现在来设计消息分发系统或者订阅者都可以，无论先做哪个，只要实现了稳定的接口，就做到了接口隔离和依赖反转，所以哪个先做都无所谓，我们先来做订阅者

---

```javascript
class Customer {
  constructor(name) {
    this.name = name;
  }

  // type TSubprops = {
  //   type: string;
  //   publisher: TPublisher;
  // }
  subscribe({ type, publisher }) {
    publisher.subscribe(type);
  }

  unsubscribe({ type, publisher }) {
    publisher.subscribe(type);
  }
}
```
