# TCP통신 N:N채팅

#### Server.java

```
package com.java.socket.lesson2;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Vector;

public class Server {
	ServerSocket ss;
	Socket socket;
	Vector<ServerThread> threads = new Vector<>();
	ServerThread thread;
	public Server(int port) {
		try {
			makerServer(port);
			while(true) {
				socket = catchSocket();
				thread = passServerData(socket, threads);
				thread.start();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public void makerServer(int port) throws IOException{
		ss = new ServerSocket(port);
		System.out.println("서버가 생성되어 연결 대기중...");
	}
	
	public Socket catchSocket() throws IOException {
		socket = ss.accept();
		System.out.println(socket+"| 클라이언트에서 받은 소켓");
		return socket;
	}
	
	public ServerThread passServerData (Socket socket, Vector<ServerThread> threads) {
		thread = new ServerThread(socket, threads);
		threads.add(thread);
		return thread;
	}
	public static void main(String[] args) {
		new Server(3000);
	}
}
```



#### ServerThread.java

```
package com.java.socket.lesson2;

import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;
import java.util.Vector;

public class ServerThread extends Thread {
	Vector<ServerThread> threads;
	Socket socket;
	DataInputStream input;
	DataOutputStream output;

	public ServerThread(Socket socket, Vector<ServerThread> threads) {
		this.socket = socket;
		this.threads = threads;
		try {
			input = connectInputStream();
			output = connectOutputStream();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public DataInputStream connectInputStream() throws IOException {  //void대신 들어간 이유는 다른 클래스에서
		System.out.println("input통로가 만들어짐"); 
		input = new DataInputStream(socket.getInputStream());         //사용할 때 훨 편함
		return input;
	}
	public DataOutputStream connectOutputStream() throws IOException {
		System.out.println("Output통로가 만들어짐");
		output = new DataOutputStream(socket.getOutputStream());
		return output;
	}
	public void sendData(String msg) throws IOException {
		System.out.println(msg+"라고 서버에 메시지를 남김");
		output.writeUTF(msg);
	}
	public String receiveData() throws IOException {
		String msg = input.readUTF();
		System.out.println(msg+"라고 받음");
		return msg;
	}
	public void broadCast(String msg) throws IOException {
		for (ServerThread thread : threads) {
			thread.sendData(msg);
		}
	}
	@Override
	public void run() {
		try {
			while(true) {
				String msg = receiveData();
				broadCast(msg);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```



#### Client.java

```
package com.java.socket.lesson2;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;

import javax.swing.JFrame;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class Client implements ActionListener{
		//선언부
		JFrame frame = new JFrame();
		JTextArea area = new JTextArea();
		JTextField text = new JTextField();
		Socket socket;
		DataInputStream input;
		DataOutputStream output;
		
		public Client(String host, int port) {
			try {
				throwSocket(host, port);
				input = connectInputStream();
				output = connectOutputStream();
				initDisplay();
				while(true) {
					String msg = receiveData();
					area.append(msg+"\n");
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		public void throwSocket(String host, int port) throws Exception{
			System.out.println("소켓을 서버쪽으로 넘기");
			socket = new Socket(host, port);
		}
		public DataInputStream connectInputStream() throws IOException {  //void대신 들어간 이유는 다른 클래스에서
			System.out.println("input통로가 만들어짐"); 
			input = new DataInputStream(socket.getInputStream());         //사용할 때 훨 편함
			return input;
		}
		public DataOutputStream connectOutputStream() throws IOException {
			System.out.println("Output통로가 만들어짐");
			output = new DataOutputStream(socket.getOutputStream());
			return output;
		}
		public void sendData(String msg) throws IOException {
			System.out.println(msg+"라고 서버에 메시지를 남김");
			output.writeUTF(msg);
		}
		public String receiveData() throws IOException {
			String msg = input.readUTF();
			System.out.println(msg+"라고 받음");
			return msg;
		}
		//메소드
		public void initDisplay() {
			text.addActionListener(this);
			area.setEditable(false);
			frame.add("Center", area);
			frame.add("South", text);
			frame.setSize(300, 500);
			frame.setVisible(true);
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		}

		
		//메인메소드
		public static void main(String[] args) {
			new Client("localhost", 3000);
		}
		
		//이벤트 처리부
		@Override
		public void actionPerformed(ActionEvent e) {
			// TODO Auto-generated method stub
			Object obj = e.getSource();
			if (obj == text) {
				String msg = text.getText();
				System.out.println(msg+"라고 들어봤어요");
				try {
					sendData(msg);
				} catch (Exception e2) {
					e2.printStackTrace();
				}
				text.setText("");
			}
		}
	}
```
