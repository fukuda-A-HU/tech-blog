---
title: "TCPソケット通信を試すpingpongサーバーの実装と、telnet, ncによる疎通確認"
emoji: "🏓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["C", "TCP", "socket", "network", "telnet", "netcat"]
published: false
---

## ゴール(はじめに)：TCPソケット通信の基礎を理解する

ネットワークプログラミングの基礎となるTCPソケット通信。多くのインターネットサービスの土台となっているこの技術について、実際に動くシンプルなサーバーを作成しながら理解を深めましょう。

この記事では、「ping」というメッセージを受け取ると「pong」と返す単純なTCPサーバーをC言語で実装し、telnetやncといったコマンドラインツールでの通信確認までを解説します。C言語でソケットプログラミングを行うことで、低レベルなネットワーク通信の仕組みを学ぶ良い入門題材になります。

## やってみた結果

この記事を実践することで次のような成果が得られます：

- TCPソケット通信の基本概念を理解できる
- C言語でシンプルなTCPサーバーを自分で実装できるようになる
- telnetやncを使ったネットワーク疎通確認の方法を学べる
- クライアント・サーバー間の通信の流れを体感できる
- POSIXソケットAPIの基本的な使い方を習得できる

実際に手を動かすことでネットワークプログラミングの基礎が身につきます。

## 開発環境

- OS: Linux, macOS, または Windows（WSL推奨）
- C コンパイラ (gcc など)
- telnet（通信確認用）
- nc（netcat、通信確認用）

フォルダ構造：
```
tcp_pingpong/
├── server.c     # 実装するTCPサーバー
└── Makefile     # ビルド用Makefile
```

## 事前準備

必要なツールがインストールされているか確認しましょう。多くのLinuxディストリビューションやmacOSでは、以下のコマンドでインストールできます：

### Ubuntuの場合
```bash
sudo apt update
sudo apt install build-essential telnet
```

### macOSの場合
```bash
xcode-select --install  # コンパイラとビルドツールをインストール
brew install telnet
```

Windowsの場合はWSL（Windows Subsystem for Linux）の利用をお勧めします。

## やったこと

シンプルなping-pongサーバーを実装し、telnetとncを使って動作確認を行います。

## TCPサーバーの実装

まず、C言語を使ってシンプルなTCPサーバーを実装します。以下のコードを`server.c`として保存してください：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <pthread.h>
#include <signal.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define HOST "127.0.0.1"  // ローカルホスト
#define PORT 8888         // ポート番号
#define BUFFER_SIZE 1024  // 受信バッファサイズ
#define BACKLOG 5         // 接続待ちキューの最大長

// クライアント接続を処理する関数のプロトタイプ宣言
void *handle_client(void *arg);

// グローバル変数
int server_fd = -1;  // サーバーソケットのファイルディスクリプタ

// シグナルハンドラ（Ctrl+Cで終了するため）
void handle_signal(int sig) {
    printf("\n[*] シグナル %d を受信。サーバーを終了します\n", sig);
    if (server_fd != -1) {
        close(server_fd);
    }
    exit(0);
}

int main() {
    struct sockaddr_in server_addr;
    
    // シグナルハンドラを設定
    signal(SIGINT, handle_signal);
    
    // ソケットの作成
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) {
        perror("[!] ソケットの作成に失敗");
        return 1;
    }
    
    // ソケットオプションの設定（アドレスの再利用を許可）
    int opt = 1;
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) < 0) {
        perror("[!] ソケットオプションの設定に失敗");
        close(server_fd);
        return 1;
    }
    
    // サーバーアドレス構造体の初期化
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(HOST);
    server_addr.sin_port = htons(PORT);
    
    // ソケットをアドレスにバインド
    if (bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("[!] バインドに失敗");
        close(server_fd);
        return 1;
    }
    
    // 接続リクエストの待ち受けを開始
    if (listen(server_fd, BACKLOG) < 0) {
        perror("[!] リッスンに失敗");
        close(server_fd);
        return 1;
    }
    
    printf("[*] サーバー起動: %s:%d\n", HOST, PORT);
    
    // クライアントからの接続を継続的に受け入れる
    while (1) {
        struct sockaddr_in client_addr;
        socklen_t client_len = sizeof(client_addr);
        
        // クライアント接続の受け入れ
        int client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_len);
        if (client_fd < 0) {
            perror("[!] 接続の受け入れに失敗");
            continue;
        }
        
        // クライアントのIPアドレスとポート番号を表示
        char client_ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client_addr.sin_addr, client_ip, sizeof(client_ip));
        printf("[*] クライアント接続: %s:%d\n", client_ip, ntohs(client_addr.sin_port));
        
        // クライアント接続を別スレッドで処理
        pthread_t thread_id;
        int *client_sock = malloc(sizeof(int));
        *client_sock = client_fd;
        
        if (pthread_create(&thread_id, NULL, handle_client, (void *)client_sock) < 0) {
            perror("[!] スレッド作成に失敗");
            close(client_fd);
            free(client_sock);
        } else {
            // スレッドをデタッチして自動的にリソースを解放
            pthread_detach(thread_id);
        }
    }
    
    // ここには到達しないが、念のため
    close(server_fd);
    return 0;
}

// クライアント接続を処理する関数
void *handle_client(void *arg) {
    int client_fd = *((int *)arg);
    free(arg);  // mallocで確保したメモリを解放
    
    char buffer[BUFFER_SIZE];
    ssize_t read_size;
    
    // 接続を維持しながらメッセージを受信
    while ((read_size = recv(client_fd, buffer, BUFFER_SIZE - 1, 0)) > 0) {
        // 受信データの終端に NULL を設定
        buffer[read_size] = '\0';
        
        // 改行文字を削除
        char *newline = strchr(buffer, '\n');
        if (newline) *newline = '\0';
        newline = strchr(buffer, '\r');
        if (newline) *newline = '\0';
        
        // 受信データを表示
        printf("[*] 受信: %s\n", buffer);
        
        // 「ping」を受け取ったら「pong」を返す
        char response[BUFFER_SIZE];
        if (strcasecmp(buffer, "ping") == 0) {
            strcpy(response, "pong\n");
        } else {
            snprintf(response, sizeof(response), "受信しました: %.1000s\n", buffer);
        }
        
        // レスポンスを送信
        printf("[*] 送信: %s", response);
        send(client_fd, response, strlen(response), 0);
    }
    
    // エラーチェック
    if (read_size == 0) {
        printf("[*] クライアントが接続を閉じました\n");
    } else if (read_size == -1) {
        perror("[!] 受信エラー");
    }
    
    // 接続を閉じる
    close(client_fd);
    printf("[*] 接続終了\n");
    
    return NULL;
}
```

## Makefileの作成

ビルドを簡単に行うために、以下の内容で`Makefile`を作成します：

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -pthread
TARGET = server
SRCS = server.c

all: $(TARGET)

$(TARGET): $(SRCS)
	$(CC) $(CFLAGS) -o $@ $<

clean:
	rm -f $(TARGET)
```

## サーバーのビルドと起動

上記のファイルを作成したら、以下のコマンドでサーバーをビルドして起動します：

```bash
# ビルド
make

# 起動
./server
```

次のような出力が表示されるはずです：

```
[*] サーバー起動: 127.0.0.1:8888
```

## telnetを使った疎通確認

別のターミナルウィンドウを開いて、telnetで接続してみましょう：

```bash
telnet 127.0.0.1 8888
```

接続後、「ping」と入力してEnterキーを押すと、「pong」という応答が返ってくるはずです：

```
$ telnet 127.0.0.1 8888
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
ping
pong
```

違うメッセージを送ると、エコーされて返ってきます：

```
hello
受信しました: hello
```

telnetセッションを終了するには、Ctrl+]を押してtelnetコマンドプロンプトに入り、「quit」と入力します。

## ncを使った疎通確認

ncコマンド（netcat）でも同様の確認ができます：

```bash
nc 127.0.0.1 8888
```

接続後は上記と同様に「ping」と入力すると「pong」が返ってきます：

```
$ nc 127.0.0.1 8888
ping
pong
```

ncセッションを終了するには、Ctrl+Cを押します。

## 負荷テスト

C言語で実装したサーバーの性能をテストするために、専用の負荷テストクライアントをC言語で作成します。これにより、Pythonのように高級言語のオーバーヘッドなしに、サーバーの処理能力を直接測定できます。

### 負荷テストのためのクライアントプログラム

以下のコードを`client_test.c`として保存します：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <sys/time.h>
#include <pthread.h>

#define HOST "127.0.0.1"
#define PORT 8888
#define MESSAGE "ping\n"
#define NUM_THREADS 10   // スレッド数

// 各スレッドの作業を定義する構造体
typedef struct {
    int thread_id;
    int requests_per_thread;
    double total_time;
    int success_count;
} thread_data;

// 時間計測用の関数
double get_time() {
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_sec + tv.tv_usec / 1000000.0;
}

// スレッド関数
void *send_requests(void *arg) {
    thread_data *data = (thread_data *)arg;
    int sock;
    struct sockaddr_in server;
    char buffer[1024];
    data->success_count = 0;
    
    double start_time = get_time();
    
    for (int i = 0; i < data->requests_per_thread; i++) {
        // ソケット作成
        sock = socket(AF_INET, SOCK_STREAM, 0);
        if (sock < 0) {
            perror("ソケット作成に失敗");
            continue;
        }
        
        // サーバー情報設定
        memset(&server, 0, sizeof(server));
        server.sin_family = AF_INET;
        server.sin_addr.s_addr = inet_addr(HOST);
        server.sin_port = htons(PORT);
        
        // サーバーに接続
        if (connect(sock, (struct sockaddr *)&server, sizeof(server)) < 0) {
            perror("接続に失敗");
            close(sock);
            continue;
        }
        
        // メッセージ送信
        if (send(sock, MESSAGE, strlen(MESSAGE), 0) < 0) {
            perror("送信に失敗");
            close(sock);
            continue;
        }
        
        // レスポンス受信
        memset(buffer, 0, sizeof(buffer));
        if (recv(sock, buffer, sizeof(buffer) - 1, 0) < 0) {
            perror("受信に失敗");
            close(sock);
            continue;
        }
        
        // 接続を閉じる
        close(sock);
        data->success_count++;
    }
    
    data->total_time = get_time() - start_time;
    
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("使用法: %s <総リクエスト数>\n", argv[0]);
        return 1;
    }
    
    int total_requests = atoi(argv[1]);
    int requests_per_thread = total_requests / NUM_THREADS;
    
    pthread_t threads[NUM_THREADS];
    thread_data thread_results[NUM_THREADS];
    
    printf("TCPサーバーに対して負荷テストを実行します...\n");
    printf("総リクエスト数: %d (スレッドあたり %d リクエスト x %d スレッド)\n", 
           total_requests, requests_per_thread, NUM_THREADS);
    
    double test_start = get_time();
    
    // スレッド作成
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_results[i].thread_id = i;
        thread_results[i].requests_per_thread = requests_per_thread;
        
        if (pthread_create(&threads[i], NULL, send_requests, &thread_results[i]) != 0) {
            perror("スレッド作成に失敗");
            return 1;
        }
    }
    
    // すべてのスレッドが完了するのを待つ
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }
    
    double test_end = get_time();
    double test_duration = test_end - test_start;
    
    // 結果の集計
    int total_success = 0;
    for (int i = 0; i < NUM_THREADS; i++) {
        total_success += thread_results[i].success_count;
    }
    
    printf("\n負荷テスト結果:\n");
    printf("総経過時間: %.2f 秒\n", test_duration);
    printf("成功リクエスト: %d / %d\n", total_success, total_requests);
    printf("リクエスト/秒: %.2f\n", total_success / test_duration);
    
    return 0;
}
```

### クライアントのビルド

`Makefile`に以下を追加して、クライアントプログラムをビルドできるようにします：

```makefile
CLIENT = client_test
CLIENT_SRCS = client_test.c

all: $(TARGET) $(CLIENT)

$(CLIENT): $(CLIENT_SRCS)
	$(CC) $(CFLAGS) -o $@ $<
```

修正した`Makefile`の全体は以下のようになります：

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -pthread
TARGET = server
SRCS = server.c
CLIENT = client_test
CLIENT_SRCS = client_test.c

all: $(TARGET) $(CLIENT)

$(TARGET): $(SRCS)
	$(CC) $(CFLAGS) -o $@ $<

$(CLIENT): $(CLIENT_SRCS)
	$(CC) $(CFLAGS) -o $@ $<

clean:
	rm -f $(TARGET) $(CLIENT)
```

### 負荷テストの実行

サーバーを起動した状態で、別のターミナルから以下のコマンドを実行します：

```bash
# 100リクエストのテスト
./client_test 100

# 1000リクエストのテスト
./client_test 1000

# 10000リクエストのテスト
./client_test 10000
```

それぞれの負荷テスト結果を比較してみましょう。以下のような結果が表示されるはずです：

```
TCPサーバーに対して負荷テストを実行します...
総リクエスト数: 10000 (スレッドあたり 1000 リクエスト x 10 スレッド)

負荷テスト結果:
総経過時間: 2.34 秒
成功リクエスト: 10000 / 10000
リクエスト/秒: 4273.50
```

C言語で実装したサーバーは、効率的な低レベル処理のため、高い処理性能を発揮することができます。ただし、実際の処理速度はマシンの性能やネットワーク環境に依存します。

### サーバーのリソース使用状況の監視

負荷テスト実行中は、別のターミナルで以下のコマンドを実行して、サーバープロセスのリソース使用状況を監視できます：

```bash
# CPUとメモリの使用率を確認
top -p $(pgrep -f "./server" | head -n 1)

# 開いているソケット接続数を確認
lsof -i :8888 | wc -l

# ネットワークの状態を確認
netstat -ant | grep 8888
```

### 高負荷時のサーバー最適化

C言語実装のサーバーでも、非常に高い負荷（秒間10000リクエスト以上）になると性能が劣化する可能性があります。以下の最適化を検討してみましょう：

1. **エフェメラルポート枯渇問題への対応**:
   - クライアント側で `SO_REUSEADDR` オプションを設定
   - TIME_WAIT 状態のソケットを減らすためのカーネルパラメータ調整

2. **接続プールの実装**:
   - 毎回新しい接続を作るのではなく、接続を再利用する仕組みを実装

3. **イベント駆動型アーキテクチャへの移行**:
   - スレッドモデルから `epoll` や `kqueue` などを使ったイベント駆動モデルへ変更
   - 高コンカレンシーに対応可能なアーキテクチャの採用

4. **ゼロコピー最適化**:
   - `sendfile()` システムコールを使用してカーネル空間とユーザー空間の間のコピーを減らす

これらの最適化は、より実用的なサーバー開発において重要になります。

## TCPソケット通信の解説

実装したコードを通じて、C言語でのTCPソケット通信の基本的な仕組みを解説します：

1. **ソケットの作成**：`socket()` 関数を使用してソケットを作成
2. **バインド**：作成したソケットをIPアドレスとポート番号にバインド（`bind()` 関数）
3. **リスニング**：接続リクエストを待ち受ける状態にする（`listen()` 関数）
4. **接続受け入れ**：クライアントからの接続リクエストを受け入れる（`accept()` 関数）
5. **データの送受信**：`recv()` と `send()` 関数を使用してデータの送受信
6. **ソケットクローズ**：`close()` 関数を使用して接続を閉じる

C言語でのソケットプログラミングはPOSIX APIに準拠しており、Unixベースのシステムで広く使用されています。Windowsでは、WinSockというMicrosoft固有の実装が使用されますが、基本的なAPIは類似しています。

## epollを使ったノンブロッキングサーバーの実装

スレッドベースのサーバーは実装が直感的ですが、接続数が増えると効率が悪くなります。そこで、Linuxの`epoll`を使ったノンブロッキングサーバー実装を紹介します。これにより、1スレッドで数千の接続を効率的に処理できるようになります。

### epollサーバーのコード

以下のコードを`server_epoll.c`として保存してください：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <signal.h>

#define MAX_EVENTS 1024    // 一度に処理するイベントの最大数
#define BUFFER_SIZE 1024   // 受信バッファサイズ
#define HOST "127.0.0.1"   // ローカルホスト
#define PORT 8888          // ポート番号
#define BACKLOG 5          // 接続待ちキューの最大長

// ソケットをノンブロッキングモードに設定する関数
static int set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}

// グローバル変数
int server_fd = -1;        // サーバーソケットのファイルディスクリプタ
int epoll_fd = -1;         // epollファイルディスクリプタ

// シグナルハンドラ（Ctrl+Cで終了するため）
void handle_signal(int sig) {
    printf("\n[*] シグナル %d を受信。サーバーを終了します\n", sig);
    if (server_fd != -1) {
        close(server_fd);
    }
    if (epoll_fd != -1) {
        close(epoll_fd);
    }
    exit(0);
}

int main() {
    struct sockaddr_in server_addr;
    struct epoll_event event, events[MAX_EVENTS];
    
    // シグナルハンドラを設定
    signal(SIGINT, handle_signal);
    
    // ソケットの作成
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    
    // ソケットオプションの設定（アドレスの再利用を許可）
    int opt = 1;
    setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    
    // サーバーアドレス構造体の初期化
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(HOST);
    server_addr.sin_port = htons(PORT);
    
    // ソケットをアドレスにバインド
    bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    
    // サーバーソケットをノンブロッキングモードに設定
    set_nonblocking(server_fd);
    
    // 接続リクエストの待ち受けを開始
    listen(server_fd, BACKLOG);
    
    printf("[*] epollサーバー起動: %s:%d\n", HOST, PORT);
    
    // epollインスタンスを作成
    epoll_fd = epoll_create1(0);
    
    // サーバーソケットをepollに登録
    event.events = EPOLLIN;  // 読み込み可能イベントを監視
    event.data.fd = server_fd;
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &event);
    
    // イベントループ
    while (1) {
        // epollイベントを待機
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        
        for (int i = 0; i < nfds; i++) {
            if (events[i].data.fd == server_fd) {
                // サーバーソケットのイベント = 新しい接続リクエスト
                struct sockaddr_in client_addr;
                socklen_t client_len = sizeof(client_addr);
                
                int client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_len);
                
                // クライアントのIPアドレスとポート番号を表示
                char client_ip[INET_ADDRSTRLEN];
                inet_ntop(AF_INET, &client_addr.sin_addr, client_ip, sizeof(client_ip));
                printf("[*] クライアント接続: %s:%d\n", client_ip, ntohs(client_addr.sin_port));
                
                // クライアントソケットをノンブロッキングモードに設定
                set_nonblocking(client_fd);
                
                // クライアントソケットをepollに登録
                event.events = EPOLLIN | EPOLLET;  // エッジトリガーモードで読み込みイベントを監視
                event.data.fd = client_fd;
                epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &event);
            }
            else {
                // クライアントソケットのイベント = データ受信またはクローズ
                int client_fd = events[i].data.fd;
                char buffer[BUFFER_SIZE];
                
                // データの読み込み
                ssize_t read_size = recv(client_fd, buffer, BUFFER_SIZE - 1, 0);
                
                if (read_size > 0) {
                    // データ受信成功
                    buffer[read_size] = '\0';
                    
                    // 改行文字を削除
                    char *newline = strchr(buffer, '\n');
                    if (newline) *newline = '\0';
                    newline = strchr(buffer, '\r');
                    if (newline) *newline = '\0';
                    
                    // 受信データを表示
                    printf("[*] 受信: %s\n", buffer);
                    
                    // 「ping」を受け取ったら「pong」を返す
                    char response[BUFFER_SIZE];
                    if (strcasecmp(buffer, "ping") == 0) {
                        strcpy(response, "pong\n");
                    } else {
                        snprintf(response, sizeof(response), "受信しました: %.1000s\n", buffer);
                    }
                    
                    // レスポンスを送信
                    printf("[*] 送信: %s", response);
                    send(client_fd, response, strlen(response), 0);
                }
                else if (read_size == 0 || (read_size == -1 && errno != EAGAIN)) {
                    // 接続クローズまたはエラー
                    printf("[*] クライアント接続終了 (fd=%d)\n", client_fd);
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, client_fd, NULL);  // epollから削除
                    close(client_fd);  // ソケットをクローズ
                }
            }
        }
    }
    
    // ここには到達しないが、念のため
    close(epoll_fd);
    close(server_fd);
    return 0;
}
```

### epollサーバーのビルドと実行

`Makefile`を更新して、epollバージョンのビルドも追加します。以下の行を追加してください：

```makefile
EPOLL_SERVER = server_epoll
EPOLL_SRCS = server_epoll.c

all: $(TARGET) $(CLIENT) $(EPOLL_SERVER)

$(EPOLL_SERVER): $(EPOLL_SRCS)
	$(CC) $(CFLAGS) -o $@ $<

clean:
	rm -f $(TARGET) $(CLIENT) $(EPOLL_SERVER)
```

更新したら、ビルドして実行します：

```bash
# ビルド
make

# 起動
./server_epoll
```

### epollサーバーの特徴

スレッドベースのサーバーと比較した、epollサーバーの主な特徴は以下の通りです：

1. **リソース効率**: 1つのスレッドで多数の接続を処理できるため、メモリ使用量が少なく、スレッド切り替えのオーバーヘッドもありません。
2. **スケーラビリティ**: 数千から数万の同時接続を処理できます。
3. **複雑さ**: イベント駆動型プログラミングは概念的に複雑で、デバッグが難しい場合があります。
4. **エッジトリガーとレベルトリガー**: epollには2つの動作モードがあり、状況に応じて選択できます。

## Locustを使った負荷テスト

負荷テストによく使われるPythonフレームワーク、Locustを使って、TCPサーバーの性能をテストする方法を紹介します。Locustは主にHTTPのテストに使用されますが、TCPソケットのテストにも応用できます。

### Locustのインストール

```bash
pip install locust
```

### TCPソケット用のLocustファイル作成

Locustを使ってTCPサーバーをテストするには、カスタムクライアントが必要です。以下のコードを`locustfile.py`として保存してください：

```python
import socket
import time
from locust import User, task, events, constant_pacing, between

# TCPクライアント
class TCPClient:
    def __init__(self, host, port):
        self.host = host
        self.port = port
    
    def send_ping(self):
        start_time = time.time()
        request_name = 'ping'
        try:
            # ソケット作成
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((self.host, self.port))
            
            # メッセージ送信
            sock.send(b'ping\n')
            
            # レスポンス受信
            response = sock.recv(1024)
            
            # 接続終了
            sock.close()
            
            # 成功イベント送信
            total_time = int((time.time() - start_time) * 1000)
            events.request.fire(
                request_type='TCP',
                name=request_name,
                response_time=total_time,
                response_length=len(response),
                exception=None
            )
            return response
        except Exception as e:
            # 失敗イベント送信
            total_time = int((time.time() - start_time) * 1000)
            events.request.fire(
                request_type='TCP',
                name=request_name,
                response_time=total_time,
                response_length=0,
                exception=e
            )
            raise e

# Locustユーザークラス
class TCPUser(User):
    abstract = True
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.client = TCPClient(self.host, 8888)  # ポート番号を指定

# pingリクエストを送信するユーザー
class PingUser(TCPUser):
    wait_time = between(0.1, 1)  # リクエスト間の待機時間（0.1〜1秒）
    
    @task
    def send_ping(self):
        self.client.send_ping()

# 継続的に高頻度でリクエストを送るユーザー
class ConstantUser(TCPUser):
    wait_time = constant_pacing(0.05)  # 50ミリ秒ごとに1リクエスト（約20 RPS/ユーザー）
    
    @task
    def send_ping(self):
        self.client.send_ping()
```

### Locustテストの実行

ターミナルで以下のコマンドを実行して、Locustを起動します：

```bash
# ホスト指定でLocustを起動
locust --host=127.0.0.1
```

Webブラウザで`http://localhost:8089`にアクセスして、テスト設定を行います：

1. Number of users：同時ユーザー数（例：100）
2. Spawn rate：1秒あたりに増加するユーザー数（例：10）
3. Host：すでに設定済み（127.0.0.1）

「Start swarming」をクリックしてテストを開始します。

### テスト結果の比較

同じパラメータでスレッドベースのサーバーとepollサーバーの両方をテストし、性能を比較します。特に以下の指標に注目します：

- **レスポンスタイム**：リクエスト処理にかかる時間
- **RPS（Requests Per Second）**：1秒あたりの処理リクエスト数
- **エラー率**：失敗したリクエストの割合
- **CPU使用率**：別ターミナルで`top`コマンドを実行して監視

### テストシナリオの例

1. **低負荷テスト**：10ユーザー、10RPS程度でサーバーの基本性能を測定
2. **中負荷テスト**：100ユーザー、100RPS程度でスケーラビリティを確認
3. **高負荷テスト**：1000ユーザー、1000RPS程度で限界性能を評価
4. **長時間耐久テスト**：中程度の負荷（100ユーザー）で30分間継続し、メモリリークや性能低下がないか確認

### 測定結果の例

典型的な測定結果は以下のようになります（実際の結果はハードウェアとネットワーク環境により異なります）：

#### スレッドベースサーバー（100ユーザー、10秒間）
- 平均レスポンスタイム: 12ms
- 最大レスポンスタイム: 145ms
- RPS: 850
- エラー率: 0.1%
- CPU使用率: 60%

#### epollサーバー（100ユーザー、10秒間）
- 平均レスポンスタイム: 8ms
- 最大レスポンスタイム: 89ms
- RPS: 1200
- エラー率: 0%
- CPU使用率: 35%

この結果から、epollサーバーはマルチスレッドサーバーよりも効率的に多数の接続を処理できることがわかります。特に同時接続数が多い場合、その差は顕著になります。

## 全体のまとめ

この記事では、以下の内容を学びました：

1. C言語での基本的なTCPソケットサーバーの実装
2. マルチスレッドモデルによるクライアント接続の同時処理
3. epollを使ったノンブロッキング・イベント駆動型サーバーの実装
4. Locustを使った負荷テストの方法
5. 異なるサーバーアーキテクチャの性能比較

ネットワークプログラミングでは、接続数や処理要件に応じて適切なアーキテクチャを選択することが重要です。少数の接続を処理する場合はスレッドモデルがシンプルで実装が容易ですが、多数の同時接続を効率的に処理するにはepollのようなイベント駆動型モデルが適しています。

これらの知識を活用して、効率的で堅牢なネットワークアプリケーションの開発に取り組んでみてください。
