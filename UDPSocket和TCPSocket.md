# UDPSocket和TCPSocket

## UDP
* UDPClient


		class UDPClient {
			    public static void main(String[] args) throws IOException, InterruptedException {
			
		        ExecutorService executorService = Executors.newCachedThreadPool();
		        while (true) {
		            executorService.execute(() -> {
		                try {
		                    InetAddress server = InetAddress.getByName("localhost");      //域名的解析
		                    byte[] bytes = ("Hello world" + Math.random()).getBytes();  //数据包
		                    DatagramPacket send = new DatagramPacket(bytes, bytes.length, server, 9999);      //指定服务器的host，端口
		                    DatagramSocket clientSocket = new DatagramSocket(); //创建socket
		                    clientSocket.send(send);      //发送数据
		
		                    DatagramPacket receiveData = new DatagramPacket(new byte[1024], 1024);  //接收数据
		                    clientSocket.receive(receiveData);    //接收数据
		                    System.out.println("client received : " + new String(receiveData.getData()));
		                } catch (Exception e) {
		
		                }
		            });
		
		            TimeUnit.MILLISECONDS.sleep(500);
		        }
		    }
		}
* UDPServer


		class UDPServer {
		    public static void main(String[] args) throws IOException {
		        DatagramSocket serverSocket = new DatagramSocket(9999);
		        while (true) {
		
		            DatagramPacket requestData = new DatagramPacket(new byte[1024], 1024);
		            serverSocket.receive(requestData);
		            System.out.println("server received :" + new String(requestData.getData()));
		            System.out.println("request host : " + requestData.getAddress().getHostAddress());
		            System.out.println("request port : " + requestData.getPort());
		
		            byte[] response = ("received your message." + new String(requestData.getData()) + requestData.getAddress().getHostAddress() + ":" + requestData.getPort()).getBytes();
		            DatagramPacket responseData = new DatagramPacket(response, response.length, requestData.getAddress(), requestData.getPort());
		            serverSocket.send(responseData);
		        }
		    }
		}

## TCP

TCPCLient

	class TCPClient {
	    public static void main(String[] args) throws IOException {
	        Socket clientSocket = null;
	        ExecutorService executorService = Executors.newCachedThreadPool();
	        clientSocket = new Socket("127.0.0.1", 9999);
	
	        Socket finalClientSocket = clientSocket;
	        try {
	            OutputStream outputStream = finalClientSocket.getOutputStream();
	            outputStream.write("Hello world".getBytes());
	
	            InputStream inputStream = finalClientSocket.getInputStream();
	            byte[] buf = new byte[1024];
	            inputStream.read(buf);
	            System.out.println("server reply : " + new String(buf));
	        } catch (Exception e) {
	
	        }
	
	    }
	}

TCPServer

	class TCPServer {
	    public static void main(String[] args) {
	        ServerSocket serverSocket = null;
	        Socket socket = null;
	        try {
	            serverSocket = new ServerSocket(9999);
	            InetAddress address = serverSocket.getInetAddress();
	            System.out.println("server host : " + address.getHostAddress());
	
	            while (true) {
	                socket = serverSocket.accept();
	                byte[] buf = new byte[1024];
	                InputStream inputStream = socket.getInputStream();
	                int cursor = inputStream.read(buf);
	                System.out.println("received : " + new String(buf, 0, cursor));
	
	                OutputStream outputStream = socket.getOutputStream();
	                outputStream.write("hello, i got you!".getBytes());
	            }
	        } catch (IOException e) {
	
	        } finally {
	            try {
	                socket.close();
	                serverSocket.close();
	            } catch (Exception e) {
	
	            }
	        }
	    }
	}		