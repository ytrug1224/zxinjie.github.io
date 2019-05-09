---
layout: post
title: Netty 源码解析--socket基础篇
categories: 网络编程
description: Socket基础介绍与用法
keywords: Java, Socket、TCP/IP
---

**TCP/IP 分层**
    网络协议通常分不同层次进行开发，每一层分别负责不同的通信功能。一个协议族，比如 TCP/IP，是一组不同层次上的多个协议的组合。 TCP/IP 通常被认为是一个四层协议系统  (此处OSI七层模型类似): 
1) 链路层，有时也称作数据链路层或网络接口层，通常包括操作系统中的设备驱动程序和计算机中对应的网络接口卡。它们一起处理与电缆（或其他任何传输媒介）的物理接口细节。
2) 网络层，有时也称作互联网层，处理分组在网络中的活动，例如分组的选路。在TCP/IP协议族中，网络层协议包括 I P协议（网际协议），I C M P协议（Internet互联网控制报文协议），以及I G M P协议（Internet组管理协议）。
3) 运输层，主要为两台主机上的应用程序提供端到端的通信。在 TCP/IP协议族中，有两个互不相同的传输协议：TCP（传输控制协议）和 UDP（用户数据报协议）。TCP为两台主机提供高可靠性的数据通信。它所做的工作包括把应用程序交给它的数据分成合适的小块交给下面的网络层，确认接收到的分组，设置发送最后确认分组的超时时钟等。由于运输层提供了高可靠性的端到端的通信，因此应用层可以忽略所有这些细节。而另一方面，UDP则为应用层提供一种非常简单的服务。它只是把称作数据报的分组从一台主机发送到另一台主机，但并不保证该数据报能到达另一端。任何必需的可靠性必须由应用层来提供。这两种运输层协议分别在不同的应用程序中有不同的用途，这一点将在后面看到。
4) 应用层，负责处理特定的应用程序细节。几乎各种不同的 TCP/IP实现都会提供下面这些通用的应用程序
• Telnet 远程登录。
• FTP 文件传输协议。
• SMTP 简单邮件传送协议。
• SNMP 简单网络管理协议。
另外还有许多其他应用，具体参看 《TCP-IP详解全卷》

**Socket**
    网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。
    建立网络通信连接至少要一对端口号(socket)。socket本质是编程接口(API)，对TCP/IP的封装，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口；HTTP是轿车，提供了封装或者显示数据的具体形式；Socket是发动机，提供了网络通信的能力。

1. TCP 方式：

服务端代码：
```
/**
 * @author zxinjie 2015/5/9
 */
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(9090);
        System.out.println("服务器打开～");
        System.out.println("服务器信息：H: " + server.getInetAddress() + " P:" + server.getLocalPort());

        // 等待客户端连接
        while (true) {
            // 得到客户端
            Socket client = server.accept();
            // 客户端构建异步线程
            ClientHandler clientHandler = new ClientHandler(client);
            // 启动线程
            clientHandler.start();
        }
    }

    /**
     * 客户端消息处理
     */
    private static class ClientHandler extends Thread {
        private Socket socket;
        private boolean flag = true;

        ClientHandler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            super.run();
            System.out.println("新客户端连接：" + socket.getInetAddress() + " P:" + socket.getPort());
            try {
                // 得到打印流，用于数据输出；服务器回送数据使用
                PrintStream socketOutput = new PrintStream(socket.getOutputStream());
                // 得到输入流，用于接收数据
                BufferedReader socketInput = new BufferedReader(new InputStreamReader(socket.getInputStream()));

                do {
                    // 客户端拿到一条数据
                    String str = socketInput.readLine();
                    if (CloseUtils.END_FLAG.equalsIgnoreCase(str)) {
                        flag = false;
                        // 回送
                        socketOutput.println(CloseUtils.END_FLAG);
                    } else {
                        // 打印到屏幕。并回送数据长度
                        System.out.println(str);
                        socketOutput.println("回送：" + str.length());
                    }
                } while (flag);

                socketInput.close();
                socketOutput.close();
            } catch (Exception e) {
                System.out.println("连接异常断开");
            } finally {
                // 连接关闭
                CloseUtils.close(socket);
            }

            System.out.println("客户端已退出：" + socket.getInetAddress() + " P:" + socket.getPort());
        }
    }
}

```

客户端：

```
/**
 * @author zxinjie 2015/5/9
 */
public class Client {

    public static void main(String[] args) throws Exception {
        Socket socket = new Socket();
        // 读取流超时时间 3s
        socket.setSoTimeout(3000);
        // 连接 、连接超时时间3s
        InetSocketAddress address = new InetSocketAddress(InetAddress.getLocalHost(), 9090);
        socket.connect(address, 3000);
        System.out.println("连接已成功!");
        System.out.println("客户端信息： H: " + socket.getLocalAddress() + ", P: " + socket.getLocalPort());
        System.out.println("服务端信息： H: " + socket.getInetAddress() + ", P: " + socket.getPort());
        try {
            sendMsg(socket);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            CloseUtils.close(socket);
        }
    }

    private static void sendMsg(Socket clientSocket) throws IOException {
        InputStream printInputStream = null;
        InputStreamReader printInputReader = null;
        BufferedReader printInputBufferReader = null;

        OutputStream outputStream = null;
        PrintStream socketSendStream = null;

        InputStream inputStream = null;
        InputStreamReader socketReader = null;
        BufferedReader socketBufferedReader = null;
        try {
            // 键盘输入流
            printInputStream = System.in;
            printInputReader = new InputStreamReader(printInputStream);
            printInputBufferReader = new BufferedReader(printInputReader);

            // 得到socket输出流，并转换为打印流
            outputStream = clientSocket.getOutputStream();
            // 发送到服务器
            socketSendStream = new PrintStream(outputStream);
            socketSendStream.flush();

            // 读取socket的返回数据
            inputStream = clientSocket.getInputStream();
            socketReader = new InputStreamReader(inputStream);
            socketBufferedReader = new BufferedReader(socketReader);

            boolean flag = true;
            do {
                String str = printInputBufferReader.readLine();
                socketSendStream.println(str);

                String echo = socketBufferedReader.readLine();
                if (CloseUtils.END_FLAG.equalsIgnoreCase(echo)) {
                    flag = false;
                } else {
                    System.out.println(echo);
                }
            } while (flag);

            CloseUtils.close(socketSendStream);
            CloseUtils.close(socketBufferedReader);
        } catch (Exception e) {
            System.out.println("异常关闭！");
        } finally {
            CloseUtils.close(outputStream);
            CloseUtils.close(inputStream);
            CloseUtils.close(clientSocket);
            System.out.println("客户端已经退出！");
        }
    }
}
```

此demo，单项数据发送返回模式；

