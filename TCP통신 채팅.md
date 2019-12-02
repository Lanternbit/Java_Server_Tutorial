# TCP통신을 이용한 채팅프로그램 만들기

### Server.java

```
package com.java.socket.lesson;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

import javax.swing.JFrame;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class Server {  //클래스 선언
	JFrame frame = new JFrame("Server");  //최상위 컨테이너 JFrame 객체 생성 (컨테이너 : 다른 컴포넌트를 안에 포함할 수 있는 컴포넌트)
	JTextArea textarea = new JTextArea();  //컴포넌트 JTextArea 객체 생성  - 여러 줄의 문자열을 입력받을 수 있는 창
	JTextField inputText = new JTextField();  //컴포넌트 JTextField 객체 생성 - 한 줄 입력창
	ServerSocket server;  //필드 생성
	int port;  //접속할 포트 번호 생성
	Socket serverSocket;  //필드 생성
	DataInputStream inputStream;
	DataOutputStream outputStream;
	
	public Server(int port) {  //만들어 놓으 메소드를 활용하여 데이터가 통신하는지 확인하기 위해 정리해 놓음
		this.port = port;
		try {
			serverSocket = makeServer(port);
			inputStream = connectInputStream();
			outputStream = connectOutputStream();
			showServerView();
			while(true) {
				String msg = receiveMessageFromClient();
				textarea.append(msg+"\n");
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	public void showServerView() {  //클래스안에  showServerView 인터페이스 구현
		textarea.setEditable(false);  //실행 시 JTextArea edit 금지
		textarea.setText("서버 화면");  //Frame에 텍스트 띄워놓기
		frame.add("Center", textarea);  //컨테이너 위에 컴포넌트 추가
		frame.add("South", inputText);  //컨테이너 위에 컴포넌트 추가
		frame.setSize(300, 500);  //Frame의 크기 설정
		frame.setVisible(true);  //생성한 Frame을 윈도우에 뿌리기
		eventHandler();
	}
	public void eventHandler() {
		inputText.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				// TODO Auto-generated method stub
				try {
					sendMessageToClient(inputText.getText());
					System.out.println("이벤트가 잘 발생했어요.");
				} catch (Exception e1) {
					// TODO: handle exception
					e1.printStackTrace();
				}
				textarea.append(inputText.getText()+"\n");
				inputText.setText("");
			}
		});
	}
	public DataInputStream connectInputStream() throws IOException {
		inputStream = new DataInputStream(serverSocket.getInputStream());
		System.out.println("데이터를 받는 통료 연결");
		return inputStream;
	}
	public DataOutputStream connectOutputStream() throws IOException {
		outputStream = new DataOutputStream(serverSocket.getOutputStream());
		System.out.println("데이터를 보내는 통로 연결");
		return outputStream;
	}
	public Socket makeServer(int port) throws IOException {  //컴파일에러륿 방지하기 위해 예외가 발생하면 해장 클래스에서 벗어나게 하는 역할
		//IOException은 Exeption의 하위클래스로 입출력시에 지정한 파일이 시스템에 존재하지 않을 때 사용
		server = new ServerSocket(port);  //port를 이용하여 ServerSocket 객체 생성
		System.out.println("연결 대기중...");
		serverSocket = server.accept();  //accept로 client가 보낸 소켓을 받음
		System.out.println("소켓을 받음");
		return serverSocket;
	}
	public void sendMessageToClient(String msg) throws IOException {  //데이터를 보내는 메서드
		outputStream.writeUTF(msg);
		System.out.println("클라이언트로 보내는 메시지");
	}
	public String receiveMessageFromClient() throws IOException {  //데이터를 받는 메서드
		String msg = inputStream.readUTF(); //서버 쪽에서 날아온 데이터를 String으로 대체해 받을 수 있다.
		System.out.println("클라이언트로부터 온 메시지");
		return msg;
	}
	public static void main(String[] args) {  //메인함수 생성
		Server server = new Server(3000);
	}

}

```


### Client.java

```
package com.java.socket.lesson;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;

import javax.swing.JFrame;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class Client {  //클래스 선언
	JFrame frame = new JFrame("Client");  //최상위 컨테이너 JFrame 객체 생성 (컨테이너 : 다른 컴포넌트를 안에 포함할 수 있는 컴포넌트)
	JTextArea textarea = new JTextArea();  //컴포넌트 JTextArea 객체 생성  - 여러 줄의 문자열을 입력받을 수 있는 창
	JTextField inputText = new JTextField();  //컴포넌트 JTextField 객체 생성 - 한 줄 입력창
	Socket clientSocket;  //필드 생성
	String host;  //필드 생성
	int port;  //접속할 포트 번호 생성
	DataOutputStream outputStream;
	DataInputStream inputStream;
	
	public Client(String host, int port) {
		this.host = host;
		this.port = port;
		try {
			clientSocket = throwSocket(host, port);
			outputStream = connectOutputStream();
			inputStream = connectInputStream(); 
			showClientView();
			while(true) {
				String msg = receiveMessageFromServer();
				textarea.append(msg+"\n");
			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	public void showClientView() {  //클래스안에  showClientView 인터페이스 구현
		textarea.setEditable(false);  //실행 시 JTextArea edit 금지
		textarea.setText("클라이언트 화면");  //Frame에 텍스트 띄워놓기
		frame.add("Center", textarea);  //컨테이너 위에 컴포넌트 추가
		frame.add("South", inputText);  //컨테이너 위에 컴포넌트 추가
		frame.setSize(300, 500);  //Frame의 크기 설정
		frame.setVisible(true);  //생성한 Frame을 윈도우에 뿌리기
		eventHandler();
	}
	public void eventHandler() {
		inputText.addActionListener(new ActionListener() {
			
			@Override
			public void actionPerformed(ActionEvent arg0) {
				// TODO Auto-generated method stub
				try {
					sendMessageToServer(inputText.getText());
					System.out.println("이벤트가 잘 발생했어요.");
				} catch (Exception e) {
					e.printStackTrace();
				}
				textarea.append(inputText.getText());
				inputText.setText("");
			}
		});
	}
	public Socket throwSocket(String host, int port) throws Exception { //소켓을 던져주는 메서드
		clientSocket = new Socket(host, port);
		System.out.println("소켓을 서버로 던짐");
		return clientSocket;  //위에 데이터형도 변경해줌, 재사용성 때문에 달아줌
	}
	public DataInputStream connectInputStream() throws IOException {  //소켓을 연결해주는 코드
		inputStream = new DataInputStream(clientSocket.getInputStream());
		System.out.println("데이터를 받는 통료 연결");
		return inputStream;
	}
	public DataOutputStream connectOutputStream() throws IOException {
		outputStream = new DataOutputStream(clientSocket.getOutputStream());
		System.out.println("데이터를 보내는 통로 연결");
		return outputStream;
	}
	public void sendMessageToServer(String msg) throws IOException {  //데이터를 보내는 메서드
		outputStream.writeUTF(msg);
		System.out.println("서버로 보내는 메시지");
	}
	public String receiveMessageFromServer() throws IOException {  //데이터를 받는 메서드
		String msg = inputStream.readUTF(); //서버 쪽에서 날아온 데이터를 String으로 대체해 받을 수 있다.
		System.out.println("서버로부터 온 메시지");
		return msg;  //msg를 메서드의 결과로 반환하고 동시에 메서드를 종료시킨다. 
	}
	public static void main(String[] args) {  //메인함수 생성
		Client client = new Client("localhost", 3000);
	}
}

```
