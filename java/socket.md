# 网络编程



## 基础概念

###  TCP/IP网络模型

1. **应用层** 负责应用程序的协议，如：HTTP,FTP,DNS
2. **传输层** 是网络程序进行通信，如：TCP,UDP
3. **网络层** 是整个TCP/IP协议的核心，用于将传输的数据进行分组，发送到目标计算机或网络，如：IP,ICMP,IGMP
4. **链路层** 用于定义物理传输通道，如：驱动程序,接口

### UDP

无连接通信协议。传输数据时，不建立逻辑连接，直接发送数据。

消耗资源小，效率高，但不保证数据完整性。

适用于丢包后影响不大的应用。

### TCP

面向连接的通信协议。先建立逻辑连接，再传输数据。

提供可靠无差错的数据传输。

### TCP的三次握手

1. 客户端向服务器发出连接请求，等待服务器确认。
2. 服务器向客户端回送一个响应，通知客户端收到连接请求。
3. 客户端再次向服务器发送确认信息，确认连接。



## InetAddress

获取自己的网络信息：

```java
InetAddress.getLocalHost();
```

获取远程网络信息：

```java
InetAddress.getAllByName("tbphp.net");
```



## UDP示例

发送端：

```java
Scanner sc = new Scanner(System.in);
DatagramSocket client = new DatagramSocket();
InetAddress ip = InetAddress.getByName("127.0.0.1");
while (true) {
    System.out.println("请输入要发送的语句：");
    byte[] cent = sc.next().getBytes();
    client.send(new DatagramPacket(cent, cent.length, ip, 7001));
}
```

接收端：

```java
DatagramSocket server = new DatagramSocket(7001);
byte[] buf = new byte[1024];
DatagramPacket packet = new DatagramPacket(buf, buf.length);
while (true) {
    server.receive(packet);
    System.out.print("[" + packet.getAddress().getHostAddress() + ":" + packet.getPort() + "]\t");
    System.out.println(new String(buf, 0, packet.getLength()));
}
```



## TCP聊天案例

Server:

```java
public class Server {

    private int port;

    private long tid;

    private ServerSocket server;

    private ArrayList<Socket> clients = new ArrayList<>();

    Server(int port) {
        this.port = port;
    }

    public void run() {
        start();
        accept();
    }

    /**
     * 开启服务
     */
    private void start() {
        try {
            server = new ServerSocket(port);
        } catch (IOException e) {
            throw new RuntimeException("开启服务失败：" + e);
        }
    }

    /**
     * 接收客户端连接
     */
    private void accept() {
        while (true) {
            try {
                task(server.accept());
            } catch (IOException e) {
                System.out.println("获取客户端连接失败：" + e);
            }
        }
    }

    /**
     * 客户端线程
     *
     * @param client 客户端
     */
    private void task(Socket client) {
        tid++;
        new Thread(() -> {
            clients.add(client);
            String clientName = "[访客" + tid + "] ";
            int len;
            byte[] b = new byte[1024];
            boolean isname = true;
            try {
                InputStream in = client.getInputStream();
                while ((len = in.read(b)) != -1) {
                    if (isname) {
                        isname = false;
                        clientName = "[" + new String(b, 0, len) + "] ";
                        System.out.println("#" + tid + " " + clientName + "连接");
                        sendAll(clientName + "进入", client);
                        continue;
                    }
                    String msg = clientName + new String(b, 0, len);
                    System.out.println("#" + tid + " " + msg);
                    sendAll(msg, client);
                }
            } catch (IOException ignored) {
            }
            System.out.println("#" + tid + " " + clientName + "断开");
            sendAll(clientName + "离开", client);
            clients.remove(client);
        }).start();
    }

    /**
     * 广播消息
     *
     * @param msg    消息内容
     * @param client 发送者
     */
    private void sendAll(String msg, Socket client) {
        byte[] b = msg.getBytes();
        int len = b.length;
        for (Socket socket : clients) {
            if (!socket.isClosed() && socket != client) {
                try {
                    socket.getOutputStream().write(b, 0, len);
                } catch (IOException e) {
                    System.out.println("广播失败：" + e);
                }
            }
        }
    }
}
```

Client:

```java
public class Client {

    private String host;

    private int port;

    private Socket client;

    Client(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() {
        connect();
        new Thread(this::read).start();
        setName();
        write();
    }

    /**
     * 设置姓名
     */
    private void setName() {
        System.out.println("请输入您的姓名：");
        try {
            client.getOutputStream().write(new Scanner(System.in).next().getBytes());
        } catch (IOException e) {
            throw new RuntimeException("姓名设置失败，断开连接");
        }
    }

    /**
     * 连接服务器
     */
    private void connect() {
        try {
            client = new Socket(host, port);
        } catch (IOException e) {
            throw new RuntimeException("连接服务器失败：" + e);
        }
    }

    /**
     * 读取信息
     */
    private void read() {
        try {
            InputStream in = client.getInputStream();
            int len;
            byte[] b = new byte[1024];
            while ((len = in.read(b)) != -1) {
                System.out.println(new String(b, 0, len));
            }
        } catch (IOException ignored) {
        }
        System.out.println("与服务器的连接已断开");
        System.exit(0);
    }

    /**
     * 发送信息
     */
    private void write() {
        try {
            OutputStream out = client.getOutputStream();
            Scanner sc = new Scanner(System.in);
            while (true) {
                out.write(sc.next().getBytes());
            }
        } catch (IOException e) {
            System.out.println("发送信息失败：" + e);
        }
    }

}
```

