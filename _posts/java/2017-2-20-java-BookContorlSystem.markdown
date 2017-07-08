---
layout: post
title: "Java编写图书馆信息管理系统"
date: 2017-2-20 14:29:07
categories: java
tags: [java, codeing]
---

这里记录的是一个图书馆信息管理系统,是之前自己闲着无聊练手的,现在记录下来就是为了回想之前的知识点,重温总结

### 运行效果

这里先贴下运行效果,进行测试

首先打开数据库

![java-bookControlSystem](/images/java/java-bookControlSystem-1.png)

之后运行main函数,如下:

<!-- more -->

#### 登录界面

![java-bookControlSystem](/images/java/java-bookControlSystem-login.png)

输出学号和密码后点击登录,就会在新的面板中出现此学号在图书馆的借阅信息,例如我输入学号为V201441122,登录后如下图

#### 操作界面

![java-bookControlSystem](/images/java/java-bookControlSystem-ui.png)

之后就可以在这个面板中进行相关的借书还书操作

代码实现的大致就这样,这里需要用到的开发工具是xampp软件包(Apache + MariaDB + PHP + Perl)作为服务器,当然里面也集成了MySQL数据库,主要的编码工具是Eclipse

知道了这些,就可以进行开发工作了

在进行开发工作时,需要对所做的系统进行规划,编写需求分析计划书和系统概要设计等等.由于这里我只是用来打发时间,所以没详细进行这编写,只是大概的绘制了系统界面功能图及流程图

#### 系统流程图

如下:

![java-bookControlSystem](/images/java/java-bookControlSystem-liuchengtu.png)

#### 界面设计图

为编码前绘制的系统登录界面图

![java-bookControlSystem](/images/java/java-bookControlSystem-plan-login-ui.png)

继而下图为信息面板:

![java-bookControlSystem](/images/java/java-bookControlSystem-plan-ms-ui.png)

依据上面的流程图和ui设计图来进行编码工作

### 编码操作

这里项目所有创建了的文件:

* ConnectionDatabase.java-连接数据库
* LoginWin.java-登录窗口
* LoginAction.java-处理登录窗口上发生的事件
* CheckAccount.java-检查验证账户的正确性及是否存在
* MsgShowWin.java-操作界面
* MsgShowWinAction.java-处理操作界面上发生的事件
* OperateDatabase.java-对数据的处理,并写入到数据库中
* MyJDialog.java-自定义的一个窗口样式
* BookControlSystem.java-程序的入口
* 数据库的建立脚本

建立需要的包和main函数后,再来创建一个文件用来连接数据库操作,例如这里为

#### ConnectionDatabase.java代码如下:

	package com.cgtest.data;

	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.SQLException;


	public class ConnectionDatabase {
	
		private static final String databaseDriver = "com.mysql.jdbc.Driver";
		private static final String databaseUrl = "jdbc:mysql://localhost:3306/bookControlSystem";
		private static final String databaseUser = "root";
		private static final String databasePasswd = "0827";
	
		public Connection conectionDatabse(){
			Connection connection = null;
		
			try {
				Class.forName(databaseDriver);
				connection = DriverManager.getConnection(databaseUrl, databaseUser, databasePasswd);
			} catch (ClassNotFoundException e) {
	//			e.printStackTrace();
				System.out.println("未找到驱动");
			} catch (SQLException e) {
	//			e.printStackTrace();
				System.out.println("e异常:未连接数据库");
	//			new MyJDialog
			}
			if(connection == null){
				System.out.println("connectionDatabase:未连接数据库");
			}
		
			return connection;
		}
	}

连接数据库需要加载驱动,所以提前在项目的根目录下建立一个lib文件夹,将数据库驱动放置文件夹内,这里的[驱动下载页面][],下载后将解压得到的jar文件就是驱动

代码中的`databaseUrl`,`databaseUser`和`databasePasswd`是自己本地利用lampp搭建的数据库服务器地址与自己配置的用户名和登录密码

这个类ConnectionDatabase在连接数据库后会返回一条连接,也就是一个Connection对象

当登录窗口运行后,当用户在该面板中产生点击操作后,程序将会连接数据库,也就是调用ConnectionDatabase类,获取一条数据库连接

如下代码登录窗口代码,文件名为

#### LoginWin.java

	package com.cgtest.ui;

	import java.awt.Dimension;
	import java.awt.Toolkit;

	import javax.swing.JButton;
	import javax.swing.JFrame;
	import javax.swing.JLabel;
	import javax.swing.JPanel;
	import javax.swing.JPasswordField;
	import javax.swing.JTextField;
	import javax.swing.SwingUtilities;
	import javax.swing.UIManager;
	import javax.swing.UnsupportedLookAndFeelException;

	import com.cgtest.action.LoginAction;

	public class LoginWin extends JFrame{
	
		private static final long serialVersionUID = 1L;
	
		private final static int loginWinWidth = 380,loginWinHeight = 270;
	
		public JTextField inputName;
		public JPasswordField inputPasswd;
		public JButton btCancel,btLogin;
	
		public LoginWin(){
			SwingUtilities.invokeLater(new Runnable() {
			
				@Override
				public void run() {
					try {
						UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
					} catch (ClassNotFoundException | InstantiationException | IllegalAccessException
							| UnsupportedLookAndFeelException e) {
						e.printStackTrace();
					}
					drawWin("华师图书馆");
				}
			});
		}
		public void drawWin(String s){
		
			JPanel panel = new JPanel();
	//		panel.setBackground(Color.yellow);
			panel.setLayout(null);
			this.add(panel);
		
			JLabel labelFirst = new JLabel("系统登陆");
			panel.add(labelFirst);
			labelFirst.setBounds(180, 10, 100, 25);
		
			JLabel labelName = new JLabel("学  号:");
			inputName = new JTextField();
			panel.add(labelName);
			panel.add(inputName);
			labelName.setBounds(90, 60, 100, 25);
			inputName.setBounds(135, 60, 150, 25);
		
			JLabel labelPasswd = new JLabel("密  码:");
			inputPasswd = new JPasswordField();
			panel.add(labelPasswd);
			panel.add(inputPasswd);
			labelPasswd.setBounds(90, 120, 100, 25);
			inputPasswd.setBounds(135, 120, 150, 25);
		
			btCancel = new JButton("取消");
			panel.add(btCancel);
			btCancel.setBounds(135, 180, 60, 22);
		
			btLogin = new JButton("登录");
			panel.add(btLogin);
			btLogin.setBounds(225, 180, 60, 22);
			btLogin.setEnabled(false);
		
			new LoginAction(this,inputName, inputPasswd, btCancel, btLogin);
		
			this.setTitle(s);
			this.setSize(loginWinWidth, loginWinHeight);
	//		this.setLayout(null);
			this.setResizable(false);
			Toolkit toolkit = Toolkit.getDefaultToolkit();
			Dimension  dimension = toolkit.getScreenSize();
			int screenWidth = dimension.width;
			int screenHeight = dimension.height;
			this.setLocation((screenWidth - loginWinWidth) / 2, (screenHeight - loginWinHeight) / 2);
			this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			this.validate();
			this.setVisible(true);
		}
	}

上面的代码主要作用是担当登录窗口的布局设置,
其中代码:

`new LoginAction(this,inputName, inputPasswd, btCancel, btLogin);`

是将使LoginWin.java和LoginAction.java连接起来,

使得登录窗口中的所有响应事件都由LoginAction.java文件来处理

构造方法中的:

`SwingUtilities.invokeLater(new Runnable())`

方法是用来获取系统主题样式,并将登录窗口的样式主题设置为于系统主题一样

方法中的参数代码:

`UIManager.getSystemLookAndFeelClassName();`

就是获取系统的主题样式

与之相对应的相应事件操作的代码如下,文件名为

#### LoginAction.java:

	package com.cgtest.action;

	import java.awt.event.ActionEvent;
	import java.awt.event.ActionListener;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;

	import javax.swing.JButton;
	import javax.swing.JFrame;
	import javax.swing.JPasswordField;
	import javax.swing.JTextField;

	import com.cgtest.data.CheckAccount;
	import com.cgtest.ui.MsgShowWin;
	import com.cgtest.ui.MyJDialog;

	public class LoginAction implements ActionListener,MouseListener{
	
		public JFrame jframe;
		public JTextField inputName;
		public JPasswordField inputPasswd;
		public JButton btCancel,btLogin;
	
		public LoginAction(JFrame jframe,JTextField inputName,JPasswordField inputPasswd,JButton btCancel,JButton btLogin){
			btCancel.addActionListener(this);
			btLogin.addActionListener(this);
			btLogin.addMouseListener(this);
			jframe.addMouseListener(this);
			inputName.addMouseListener(this);
			inputPasswd.addMouseListener(this);
			this.jframe = jframe;
			this.inputName = inputName;
			this.inputPasswd = inputPasswd;
			this.btCancel = btCancel;
			this.btLogin = btLogin;
		}

		@Override
		public void actionPerformed(ActionEvent e) {
			String strInputName = inputName.getText(),strInputPasswd = String.valueOf(inputPasswd.getPassword());//密码不能直接toString()
			CheckAccount checkAccount = new CheckAccount();
			int checkConnect = checkAccount.checkConnectDatabase();
			if(e.getSource() == btCancel){
				new MyJDialog(true, jframe).showChoice("确定退出?");
			
			}else if(e.getSource() == btLogin){
				int n = checkAccount.checkNamePasswd(strInputName, strInputPasswd);
				if(checkConnect == 1){
					if(n == -1){
						new MyJDialog(true, jframe).showMessage("无数据");
					}else if(n == -2){
						new MyJDialog(true, jframe).showMessage("用户不存在");
					}else if(n == -3){
						new MyJDialog(true, jframe).showMessage("密码错误,请重新输入");
					}else if(n == 1){
						new MsgShowWin(checkAccount);
						jframe.dispose();
					}else{
						System.out.println("代码错误");
					}
				}else{
					new MyJDialog(true, jframe).showMessage("未连接数据库");
				}
			}
		}

		@Override
		public void mouseClicked(MouseEvent e) {
		
		}

		@Override
		public void mousePressed(MouseEvent e) {
		
		}

		@Override
		public void mouseReleased(MouseEvent e) {
		
		}

		@Override
		public void mouseEntered(MouseEvent e) {
			// TODO Auto-generated method stub
			if(e.getSource() == jframe){
				if(inputName.getText().length() != 0 && inputPasswd.getPassword().length != 0){
					btLogin.setEnabled(true);
				}else{
					btLogin.setEnabled(false);
				}
			}
		}

		@Override
		public void mouseExited(MouseEvent e) {
			// TODO Auto-generated method stub
			if(e.getSource() == inputName || e.getSource() == inputPasswd){
				if(inputName.getText().length() != 0 && inputPasswd.getPassword().length != 0){
					btLogin.setEnabled(true);
				}else{
					btLogin.setEnabled(false);
				}
			}
		}
	}

上面代码中actionPerformed(ActionEvent e)方法中,

代码:

`CheckAccount checkAccount = new CheckAccount();`

将产生一个数据库连接

代码:

`int checkConnect = checkAccount.checkConnectDatabase();`

中的checkConnectDatabase\(\)方法将是用来检查数据库服务器是否开启,如果是,则进行下一步:验证用户输入的用户名是否存在.就如下代码

代码:

`int n = checkAccount.checkNamePasswd(strInputName, strInputPasswd);`

方法chexkNamePasswd\(strInputName, strInputPasswd\)将用户输入的用户名和密码作为参数,用来验证是否存在或正确

因此,来看对象checkAccount的如下代码,文件名为

#### CheckAccount.java:

	package com.cgtest.data;

	import java.sql.Connection;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.util.Vector;


	public class CheckAccount {
	
		public Vector<Object> vectorUserInputLogined = new Vector<Object>();
	
		public CheckAccount(){
		
		}
		public Vector<Vector<Object>> getAllNamePasswd(){
			/*
			 * 获取并返回数据库中所有的用户名和密码
			 */
			Vector<Vector<Object>> vectorAllNamePasswd = new Vector<>();
			String sqlGetNamePasswd = "SELECT accountName,accountPasswd FROM accountMsg";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(sqlGetNamePasswd);
				while(resultSet.next()){
					Vector<Object> vectorOne = new Vector<Object>();
					for(int i = 1; i <= 2; i++){
						vectorOne.add(resultSet.getString(i));
					}
					vectorAllNamePasswd.add(vectorOne);
				}
				resultSet.close();
			} catch (SQLException e) {
				e.printStackTrace();
	//			System.out.println("未连接数据库");
			}
			try {
			
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return vectorAllNamePasswd;
		}
	
		public int checkNamePasswd(String strInputName, String strInputPasswd){
			/*
			 * 根据用户输入的用户名和密码,并以此来对比数据库中存在的用户名和密码,验证登录
			 */
			int n = 0, index = 0;
			Vector<Vector<Object>> vectorAllNamePasswd = getAllNamePasswd();
			if(vectorAllNamePasswd.size() == 0){
	//			new MyJDialog(true, jframe).showMessage("未连接数据库");
				n = -1;
			}else{
				for(int i = 0; i < vectorAllNamePasswd.size(); i++){
					if(vectorAllNamePasswd.elementAt(i).get(0).equals(strInputName)){
						index = i;
						break;
					}else{
						index = -1;
					}
				}
				if(index == -1){
	//				new MyJDialog(true, jframe).showMessage("用户不存在");
					n = -2;
				}else{
					if(vectorAllNamePasswd.elementAt(index).get(1).equals(strInputPasswd)){
	//					new MsgShowWin();
						vectorUserInputLogined.add(vectorAllNamePasswd.elementAt(index).get(0));
						vectorUserInputLogined.add(vectorAllNamePasswd.elementAt(index).get(1));
						/*
						 * 只有当账户密码都输入正确后(成功登录后),才将登录账户发送到vectorUserInputLogined
						 */
						n = 1;
					}else{
	//					new MyJDialog(true, jframe).showMessage("密码错误,请重新输入");
						n = -3;
					}
				}
			}
	//		if(vectorAllNamePasswd.size() == 0){
	//			System.out.println("空");
	//		}else{
	//			System.out.println("非空" + index);
	//		}
			return n;
		}
	
		public Vector<Object> transmitAccountMsg(){
			/*
			 * 只有当用户成功登录后,才返回登录账户密码
			 */
			Vector<Object> vector = new Vector<Object>();
	//		System.out.println(vectorUserInputLogined.size());
			if(this.vectorUserInputLogined.size() != 0){
				for(int i = 0; i < this.vectorUserInputLogined.size(); i++){
	//				System.out.println(vectorUserInputLogined.get(i));
					vector.add(this.vectorUserInputLogined.get(i));
				}
			}else{
	//			System.out.println("vectorUserInputLogined:用户未登录");
			}
			return vector;
		}
	
	
		public int checkConnectDatabase(){
			/*
			 * for loginWin登录窗口
			 */
			int n = 0;
			Connection connection = new ConnectionDatabase().conectionDatabse();
			if(connection != null){
				n = 1;
			}else{ 
				n = -1;
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return n;
		}
	}

看上面的代码,这里主要说方法checkNamePasswd\(String strInputName, String strInputPasswd\)的工作方式

首相checkNamePasswd\(\)方法将调用自身类的一个getAllNamePasswd\(\)无参数方法,用来获取数据库中所有已注册的用户账户名和密码,getAllNamePasswd\()的返回值为`Vector<Vector<Object>>`双层的Vector集合,所以会返回一条双层的
Vector集合,返回的集合将用在checkNamePasswd方法中作为局部变量
方法checkNamePasswd()将先根据用户输入的用户名来和数据库中所有存在的用户名来对比,如果为对比到,那么将返回-2,意味着用户不存在

方法checkNamePasswd()将会有多中int类型的返回值,其返回值将用在LoginAction类中的actionPerformed\(ActionEvent e\)中,将根据其返回值来判断是否存在/正确,如果返回值为1,那么证明用户输入的用户名和密码都正确,则将打开下一个信息面板,而关闭登录窗口.再将登录的账户名和密码封装进一个专有的Vector集合,并返回

返回的Vector集合将会通过LoginAction类利用代码`new MsgShowWin(checkAccount);`来传递给MsgShowWin这个类,使得在打开这个面板的时候就可以直接通过账户名来获取借阅信息并显示在信息面板上

如下信息窗口代码,文件名为

#### MsgShowWin.java:

	package com.cgtest.ui;

	import java.awt.Color;
	import java.awt.Dimension;
	import java.awt.Toolkit;
	import java.util.Vector;

	import javax.swing.BorderFactory;
	import javax.swing.JButton;
	import javax.swing.JFrame;
	import javax.swing.JLabel;
	import javax.swing.JMenu;
	import javax.swing.JMenuBar;
	import javax.swing.JMenuItem;
	import javax.swing.JPanel;
	import javax.swing.JScrollPane;
	import javax.swing.JTable;
	import javax.swing.JTextField;
	import javax.swing.SwingUtilities;
	import javax.swing.UIManager;
	import javax.swing.UnsupportedLookAndFeelException;
	import javax.swing.border.MatteBorder;
	import javax.swing.table.DefaultTableCellRenderer;
	import javax.swing.table.DefaultTableModel;

	import com.cgtest.action.MsgShowWinAction;
	import com.cgtest.data.CheckAccount;
	import com.cgtest.data.OperateDatabase;

	public class MsgShowWin extends JFrame{
	
		private static final long serialVersionUID = 1L;
	
		private final static int msgWinWidth = 800,msgWinHeight = 500;
	
		public String accountName, personName, sex, allBookNum ,strShowDBMsg;
	
		public JLabel jlabelAccountName, jlabelPersonName, jlabelSex, jlabelAllBookNum, jlabelShowDBMsg;
		public JPanel showPortrait;
		public JTextField jtfInputIndexBorrow,jtfShowBookBorrow,jtfInputIndexReturn,jtfShowBookReturn;
		public JButton btSearchBorrow,btConfirmBorrow,btSearchReturn,btConfirmReturn;
		public DefaultTableModel defaultTableModel;
		public JTable jtableMsg;
		public Vector<Vector<Object>> vectorJTableMsg;
		public Vector<Object> vectorJTableTitle;
		public JMenuBar jmenuBar;
		public JMenu jmenu;
		public JMenuItem jmenuItemLogout, jmenuItemExit;
	
		public int myJDialogChoice;
	
		public MsgShowWin(CheckAccount checkAccount){
			SwingUtilities.invokeLater(new Runnable() {
			
				@Override
				public void run() {
					try {
						UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
					} catch (ClassNotFoundException | InstantiationException | IllegalAccessException
							| UnsupportedLookAndFeelException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					strShowDBMsg = new OperateDatabase().checkConnectDatabase();
					setShowAccountMsg(checkAccount);
					vectorJTableMsg = new OperateDatabase().getUserBorrowMsg(checkAccount);
					int bookNum = vectorJTableMsg.size();
					allBookNum = bookNum + "/10";
					drawMsgShowWin();
				}
			});
		}
	
		public void drawMsgShowWin(){
		
			jmenuBar = new JMenuBar();
			this.setJMenuBar(jmenuBar);
		
			jmenu = new JMenu("选项");
			jmenuBar.add(jmenu);
		
			jmenuItemLogout = new JMenuItem("注销");
			jmenuItemExit = new JMenuItem("退出");
			jmenu.add(jmenuItemLogout);
			jmenu.add(jmenuItemExit);
		
		
			JPanel jpanelMsg = new JPanel();
			JLabel jlabel1 = new JLabel("学号:");
			JLabel jlabel2 = new JLabel("姓名:");
			JLabel jlabel3 = new JLabel("性别:");
			JLabel jlabel4 = new JLabel("可借数量:");
		
			this.setLayout(null);
		
			JPanel panelWest = new JPanel();
			JPanel panelEast = new JPanel();
			JPanel panelSouth = new JPanel();
			panelWest.setBackground(new Color(255, 255, 243));
			panelEast.setBackground(new Color(255, 255, 243));
			panelSouth.setBackground(new Color(255, 255, 243));
			panelWest.setBounds(0, 0, 396, 180);
			panelEast.setBounds(396, 0, 400, 180);
			panelSouth.setBounds(0, 180, 800, 280);
			panelWest.setLayout(null);
			panelEast.setLayout(null);
			panelSouth.setLayout(null);
			this.add(panelWest);
			this.add(panelEast);
			this.add(panelSouth);
		
			panelWest.add(jpanelMsg);
			jpanelMsg.setBounds(47, 5, 160, 165);
			jpanelMsg.setLayout(null);
			jpanelMsg.setBackground(new Color(255,255,255));
			jpanelMsg.setBorder(new MatteBorder(0,0,0,1,Color.gray));
		
			jpanelMsg.add(jlabel1);
			jlabel1.setBounds(0, 15, 30, 25);
			jlabelAccountName = new JLabel(accountName);
			jpanelMsg.add(jlabelAccountName);
			jlabelAccountName.setBounds(35, 15, 90, 25);
			jlabelAccountName.setOpaque(true);
			jlabelAccountName.setBackground(new Color(255,255,255));
		
			jpanelMsg.add(jlabel2);
			jlabel2.setBounds(0, 48, 30, 25);
			jlabelPersonName = new JLabel(personName);
			jpanelMsg.add(jlabelPersonName);
			jlabelPersonName.setBounds(35, 48, 70, 25);
			jlabelPersonName.setOpaque(true);
			jlabelPersonName.setBackground(new Color(255,255,255));
		
			jpanelMsg.add(jlabel3);
			jlabel3.setBounds(0, 81, 30, 25);
			jlabelSex = new JLabel(sex);
			jpanelMsg.add(jlabelSex);
			jlabelSex.setBounds(35, 81, 30, 25);
			jlabelSex.setOpaque(true);
			jlabelSex.setBackground(new Color(255,255,255));
		
			jpanelMsg.add(jlabel4);
			jlabel4.setBounds(0, 114, 60, 25);
			jlabelAllBookNum = new JLabel(allBookNum);
			jpanelMsg.add(jlabelAllBookNum);
			jlabelAllBookNum.setBounds(65, 114, 60, 25);
			jlabelAllBookNum.setOpaque(true);
			jlabelAllBookNum.setBackground(new Color(255,255,255));
		
			showPortrait = new JPanel();
			panelWest.add(showPortrait);
			showPortrait.setBounds(240, 16, 120, 135);
			showPortrait.setBorder(BorderFactory.createLineBorder(Color.gray));
		
			JPanel jpanelChoice1 = new JPanel();
			JPanel jpanelChoice2 = new JPanel();
		
			panelEast.add(jpanelChoice1);
			panelEast.add(jpanelChoice2);
			jpanelChoice1.setLayout(null);
			jpanelChoice2.setLayout(null);
			jpanelChoice1.setBounds(0, 0, 386, 90);
			jpanelChoice2.setBounds(0, 90, 386, 90);
			jpanelChoice1.setBackground(new Color(255, 255, 243));
			jpanelChoice2.setBackground(new Color(255, 255, 243));
			jpanelChoice1.setBorder(new MatteBorder(0, 1, 1, 0, Color.gray));
			jpanelChoice2.setBorder(new MatteBorder(0, 1, 0, 0, Color.gray));
		
			JLabel jlabel5 = new JLabel("借书");
			JLabel jlabel6 = new JLabel("还书");
		
			jpanelChoice1.add(jlabel5);
			jlabel5.setBounds(30, 31, 30, 25);
		
			jtfInputIndexBorrow = new JTextField();
			jtfInputIndexBorrow.setText("输入所借图书的索引");
			jpanelChoice1.add(jtfInputIndexBorrow);
			jtfInputIndexBorrow.setBounds(80, 10, 200, 26);
		
			btSearchBorrow = new JButton("查找");
			jpanelChoice1.add(btSearchBorrow);
			btSearchBorrow.setBounds(290,11,60,25);
		
			jtfShowBookBorrow = new JTextField();
			jtfShowBookBorrow.setEnabled(false);
			jpanelChoice1.add(jtfShowBookBorrow);
			jtfShowBookBorrow.setBounds(80, 50, 200, 26);
		
			btConfirmBorrow = new JButton("借书");
			btConfirmBorrow.setEnabled(false);
			jpanelChoice1.add(btConfirmBorrow);
			btConfirmBorrow.setBounds(290,51,60,25);
		
			jpanelChoice2.add(jlabel6);
			jlabel6.setBounds(30, 31, 30, 25);
		
			jtfInputIndexReturn = new JTextField();
			jtfInputIndexReturn.setText("输入所还图书的索引");
			jpanelChoice2.add(jtfInputIndexReturn);
			jtfInputIndexReturn.setBounds(80, 10, 200, 26);
		
			btSearchReturn = new JButton("查找");
			jpanelChoice2.add(btSearchReturn);
			btSearchReturn.setBounds(290,11,60,25);
		
			jtfShowBookReturn = new JTextField();
			jtfShowBookReturn.setEnabled(false);
			jpanelChoice2.add(jtfShowBookReturn);
			jtfShowBookReturn.setBounds(80, 50, 200, 26);
		
			btConfirmReturn = new JButton("还书");
			btConfirmReturn.setEnabled(false);
			jpanelChoice2.add(btConfirmReturn);
			btConfirmReturn.setBounds(290,51,60,25);
		
			vectorJTableTitle = new Vector<Object>();
		
			vectorJTableTitle.add(0, "索引");
			vectorJTableTitle.add(1, "书名");
			vectorJTableTitle.add(2, "借书日期");
			vectorJTableTitle.add(3, "到期时间");
	
			JPanel jpanelJTable = new JPanel();
			panelSouth.add(jpanelJTable);
			jpanelJTable.setLayout(null);
			jpanelJTable.setBounds(15, 0, 768, 255);
			jpanelJTable.setBackground(new Color(255, 255, 243));
			jpanelJTable.setBorder(BorderFactory.createTitledBorder(new MatteBorder(1, 0, 0, 0, Color.gray), "借书详情"));
		
			defaultTableModel = new DefaultTableModel(vectorJTableMsg, vectorJTableTitle);
			jtableMsg = new JTable(defaultTableModel);
			jtableMsg.setRowHeight(25);
			DefaultTableCellRenderer r = new DefaultTableCellRenderer();   
			r.setHorizontalAlignment(JLabel.CENTER);   
			jtableMsg.setDefaultRenderer(Object.class,r);
			JScrollPane jscrollPane = new JScrollPane(jtableMsg);
			jpanelJTable.add(jscrollPane);
			jscrollPane.setBackground(Color.white);
			jscrollPane.setBounds(0, 18, 765, 235);
		
			jlabelShowDBMsg = new JLabel(strShowDBMsg);
			panelSouth.add(jlabelShowDBMsg);
			jlabelShowDBMsg.setBounds(680, 250, 100, 20);
		
		
			new MsgShowWinAction(this, jmenuItemLogout, jmenuItemExit, btSearchBorrow, btConfirmBorrow, btSearchReturn,btConfirmReturn,
					jtfInputIndexBorrow, jtfShowBookBorrow, jtfInputIndexReturn, jtfShowBookReturn, jtableMsg, vectorJTableTitle, 
					accountName, jlabelAllBookNum);
		
			this.setTitle("华师图书馆");
			this.setSize(msgWinWidth, msgWinHeight);
			Toolkit toolkit = Toolkit.getDefaultToolkit();
			Dimension dimension = toolkit.getScreenSize();
			int screenWidth = dimension.width;
			int screenHeight = dimension.height;
			this.setLocation((screenWidth - msgWinWidth) / 2, (screenHeight - msgWinHeight) / 2);
			this.setResizable(false);
			this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			this.validate();
			this.setVisible(true);
		}
	
		public void setShowAccountMsg(CheckAccount checkAccount){
			Vector<Object> vectorAccountMsg = new OperateDatabase().getAccountMsg(checkAccount);
			accountName = String.valueOf(vectorAccountMsg.get(0));
			personName = String.valueOf(vectorAccountMsg.get(2));
			sex = String.valueOf(vectorAccountMsg.get(3));
		}
	}

同登录窗口的LoginWin.java文件一样,上面代码的大体作用一样是面板的布局

面板第一时间显示的信息都在该类的构造方法中完成

例如代码:

`strShowDBMsg = new OperateDatabase().checkConnectDatabase();`

将调用一个OperateDatabase类中的checkConnectDatabase\(\)方法来检测是否与数据库进行连接,其返回值为String类型的数据,用来在该面板底部显示数据库连接情况.起初并未打算在此处检测是否连接,但貌似那会儿没想到如何用来传递这种信息,所以就在这进行检测了

后面的几行代码依然如此

下面贴下于MsgShowWin类对应的action时间处理类的代码,文件名

#### MsgShowWinAction.java:

	package com.cgtest.action;

	import java.awt.event.ActionEvent;
	import java.awt.event.ActionListener;
	import java.awt.event.FocusEvent;
	import java.awt.event.FocusListener;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.util.Vector;

	import javax.swing.JButton;
	import javax.swing.JFrame;
	import javax.swing.JLabel;
	import javax.swing.JMenuItem;
	import javax.swing.JTable;
	import javax.swing.JTextField;
	import javax.swing.table.DefaultTableModel;

	import com.cgtest.data.OperateDatabase;
	import com.cgtest.ui.MyJDialog;

	public class MsgShowWinAction implements ActionListener, MouseListener, FocusListener{
	
		public JFrame jframe;
		public JMenuItem jmenuItemLogout,jmenuItemExit;
		public JButton btSearchBorrow,btConfirmBorrow,btSearchReturn,btConfirmReturn;
		public JTextField jtfInputIndexBorrow,jtfShowBookBorrow,jtfInputIndexReturn,jtfShowBookReturn;
		public JTable jtable;
		public Vector<Object> vectorJTableTitle;
		public String accountName;
		public JLabel jlabelAllBookNum;
	
	
		public MsgShowWinAction(JFrame jframe, JMenuItem jmenuItemLogout, JMenuItem jmenuItemExit, JButton btSearchBorrow, JButton btConfirmBorrow,
				JButton btSearchReturn, JButton btConfirmReturn, JTextField jtfInputIndexBorrow, JTextField jtfShowBookBorrow,
				JTextField jtfInputIndexReturn, JTextField jtfShowBookReturn, JTable jtable, Vector<Object> vectorJTableTitle, 
				String accountName, JLabel jlabelAllBookNum){
			this.jframe = jframe;
			this.jmenuItemLogout = jmenuItemLogout;
			this.jmenuItemExit = jmenuItemExit;
			this.btSearchBorrow = btSearchBorrow;
			this.btConfirmBorrow = btConfirmBorrow;
			this.btSearchReturn = btSearchReturn;
			this.btConfirmReturn = btConfirmReturn;
			this.jtfInputIndexBorrow = jtfInputIndexBorrow;
			this.jtfShowBookBorrow = jtfShowBookBorrow;
			this.jtfInputIndexReturn = jtfInputIndexReturn;
			this.jtfShowBookReturn = jtfShowBookReturn;
			this.jtable = jtable;
			this.vectorJTableTitle = vectorJTableTitle;
			this.accountName = accountName;
			this.jlabelAllBookNum = jlabelAllBookNum;
			jmenuItemLogout.addActionListener(this);
			jmenuItemExit.addActionListener(this);
			btSearchBorrow.addActionListener(this);
			btConfirmBorrow.addActionListener(this);
			btSearchReturn.addActionListener(this);
			btConfirmReturn.addActionListener(this);
			jframe.addMouseListener(this);
			jtfShowBookBorrow.addMouseListener(this);
			jtfShowBookReturn.addMouseListener(this);
			btConfirmBorrow.addMouseListener(this);
			btConfirmReturn.addMouseListener(this);
			jtfInputIndexBorrow.addFocusListener(this);
			jtfInputIndexReturn.addFocusListener(this);
		}

		@Override
		public void actionPerformed(ActionEvent e) {
			if(e.getSource() == jmenuItemLogout){
			
				MyJDialog myJDialog = new MyJDialog(true, jframe);
				myJDialog.showChoiceForMsgWin("确定注销登录吗?");
			
			}else if(e.getSource() == jmenuItemExit){
			
				new MyJDialog(true, jframe).showChoice("确定退出系统?");
			
			}else if(e.getSource() == btSearchBorrow){
			
				String strjtfInputIndexBorrow = jtfInputIndexBorrow.getText();
				if(strjtfInputIndexBorrow.length() != 0){
					String strMsgorName = new OperateDatabase().searchBook(strjtfInputIndexBorrow);
					if(strMsgorName.equals("未找到此书")){
						new MyJDialog(true, jframe).showMessage("未找到此书");
					}else{
						jtfShowBookBorrow.setText(strMsgorName);
					}
				}else{
					new MyJDialog(true, jframe).showMessage("请输入图书索引");
				}
			
			}else if(e.getSource() == btConfirmBorrow){
				Vector<Vector<Object>> vectorJTableMsg = new Vector<Vector<Object>>();
	//			System.out.println(accountName);
				String strjtfInputIndexBorrow = jtfInputIndexBorrow.getText();
				String strjtfShowBookBorrow = jtfShowBookBorrow.getText();
				String str = new OperateDatabase().borrowBook(strjtfInputIndexBorrow, strjtfShowBookBorrow, accountName);
				new MyJDialog(true, jframe).showMessage(str);
				if(str.equals("借书成功")){
					vectorJTableMsg = new OperateDatabase().getUserBorrowMsg(accountName);
					int bookNumNew = vectorJTableMsg.size();
					DefaultTableModel defaultTableModel = new DefaultTableModel(vectorJTableMsg, vectorJTableTitle);
					jtable.setModel(defaultTableModel);
					String allBookNumNew = bookNumNew + "/10";
					jlabelAllBookNum.setText(allBookNumNew);
				}
			}else if(e.getSource() == btSearchReturn){
			
				String strjtfInputIndexReturn = jtfInputIndexReturn.getText();
				if(strjtfInputIndexReturn.length() != 0){
					String strMsgorName = new OperateDatabase().searchBook(strjtfInputIndexReturn);
					if(strMsgorName.equals("未找到此书")){
						new MyJDialog(true, jframe).showMessage("未找到此书");
					}else{
						jtfShowBookReturn.setText(strMsgorName);
					}
				}else{
					new MyJDialog(true, jframe).showMessage("请输入图书索引");
				}
			
			}else if(e.getSource() == btConfirmReturn){
				Vector<Vector<Object>> vectorJTableMsg = new Vector<Vector<Object>>();
				String strjtfShowBookReturn = jtfShowBookReturn.getText();
				String str = new OperateDatabase().returnBook(strjtfShowBookReturn);
				new MyJDialog(true, jframe).showMessage(str);
				if(str.equals("还书成功")){
					vectorJTableMsg = new OperateDatabase().getUserBorrowMsg(accountName);
					int bookNumNew = vectorJTableMsg.size();
					DefaultTableModel defaultTableModel = new DefaultTableModel(vectorJTableMsg, vectorJTableTitle);
					jtable.setModel(defaultTableModel);
					String allBookNumNew = bookNumNew + "/10";
					jlabelAllBookNum.setText(allBookNumNew);
				}
			}
		}

		@Override
		public void mouseClicked(MouseEvent e) {
		
		}

		@Override
		public void mousePressed(MouseEvent e) {
		
		}

		@Override
		public void mouseReleased(MouseEvent e) {
		
		}

		@Override
		public void mouseEntered(MouseEvent e) {
			// TODO Auto-generated method stub
			if(e.getSource() == btConfirmBorrow){
				if(jtfShowBookBorrow.getText().length() != 0){
					btConfirmBorrow.setEnabled(true);
				}else{
					btConfirmBorrow.setEnabled(false);
				}
			}else if(e.getSource() == btConfirmReturn){
				if(jtfShowBookReturn.getText().length() != 0){
					btConfirmReturn.setEnabled(true);
				}else{
					btConfirmReturn.setEnabled(false);
				}
			}else if(e.getSource() == jframe){
				if(jtfShowBookBorrow.getText().length() != 0){
					btConfirmBorrow.setEnabled(true);
				}else if(jtfShowBookBorrow.getText().length() == 0){
					btConfirmBorrow.setEnabled(false);
				}else if(jtfShowBookReturn.getText().length() != 0){
					btConfirmReturn.setEnabled(true);
				}else if(jtfShowBookReturn.getText().length() == 0){
					btConfirmReturn.setEnabled(false);
				}
			}
		}

		@Override
		public void mouseExited(MouseEvent e) {
		
		}

		@Override
		public void focusGained(FocusEvent e) {
			// TODO Auto-generated method stub
			if(e.getSource() == jtfInputIndexBorrow){
				if(jtfInputIndexBorrow.getText().equals("输入所借图书的索引")){
					jtfInputIndexBorrow.setText(null);
				}
				jtfInputIndexReturn.setText("输入所还图书的索引");
				jtfShowBookBorrow.setText(null);
				jtfShowBookReturn.setText(null);
			}else if(e.getSource() == jtfInputIndexReturn){
				if(jtfInputIndexReturn.getText().equals("输入所还图书的索引")){
					jtfInputIndexReturn.setText(null);
				}
				jtfInputIndexBorrow.setText("输入所借图书的索引");
				jtfShowBookBorrow.setText(null);
				jtfShowBookReturn.setText(null);
			}
		}

		@Override
		public void focusLost(FocusEvent e) {
			// TODO Auto-generated method stub
			if(e.getSource() == jtfInputIndexBorrow){
				if(jtfInputIndexBorrow.getText().length() == 0){
					jtfInputIndexBorrow.setText("输入所借图书的索引");
				}
			}else if(e.getSource() == jtfInputIndexReturn){
				if(jtfInputIndexReturn.getText().length() == 0){
				
					jtfInputIndexReturn.setText("输入所还图书的索引");
				}
			}
		}

	}

上面的代码中,在actionPerformed(ActionEvent e)方法中就是执行借书/还书/查询的操作

这里就直接看下实现借书操作的代码片段

借书分为两步,先是查找书籍,再是借书

先看查书,实现搜索图书的关键方法代码:

`searchBook(strjtfInputIndexBorrow);`

这里将用户输入的图书索引用作OperateDatabase类中的searchBook(strjtfInputIndexBorrow)的参数,

其searchBook(strjtfInputIndexBorrow)的返回值为String类型的,且只有两种返回值,

其返回值赋值给strMsgorName

从此行的下一行代码`if(strMsgorName.equals("未找到此书")){`就可以看出

再看借书,代码是调用OperateDataBase中的borrowBook(strjtfInputIndexBorrow, strjtfShowBookBorrow, accountName)方法

类OperateDataBase中的borrowBook\(\)方法有3个参数,

`strjtfInputIndexBorrow`即为查找图书用的索引,查书不到就无法进行借书,所以查找图书所用的索引是正确的

`strjtfShowBookBorrow`即图书名字

`accountName`及账户名

方法borrowBook()的返回值也为String类型,且只有两种情况

同样根据下面一行代码`if(str.equals("借书成功")){`就可以确认,其跟searchBook(strjtfInputIndexBorrow)类似

再来看类OperateDatabase的代码,文件名为

#### OperateDatabase.java:

	package com.cgtest.data;

	import java.sql.Connection;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.text.SimpleDateFormat;
	import java.util.Calendar;
	import java.util.Date;
	import java.util.Vector;

	public class OperateDatabase {
		public Vector<Object> dataAccount= new Vector<>();
	
		public Vector<Object> getAccountMsg(CheckAccount checkAccount){
			/*
			 * 登录前(打开MsgShowWin面板前)获取账户信息
			 * 使用原先创建CheckAccount对象的引用,来传递checkAccount的引用的vector
			 * 依据登录名来获取姓名,性别
			 */
			Vector<Object> vectorAccountMsg = new Vector<Object>();
			dataAccount = checkAccount.transmitAccountMsg();
	//		if(dataAccount.size() == 0){
	//			System.out.println("dataAccount:未得到数据");
	//		}else{
	//			for(int i = 0; i < dataAccount.size(); i++){
	//				System.out.println(dataAccount.get(i));
	//			}
	//		}
			String strAccountName = String.valueOf(dataAccount.get(0));
			String strSearchAccountMsg = "SELECT * FROM accountMsg WHERE accountName = " + "'" + strAccountName + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(strSearchAccountMsg);
				while(resultSet.next()){
					for(int i = 1; i <= 6; i++){
						vectorAccountMsg.add(resultSet.getString(i));
					}
				}
				resultSet.close();
				statement.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		
			return vectorAccountMsg;
		}
	
		public Vector<Vector<Object>> getUserBorrowMsg(CheckAccount checkAccount){
			/*
			 * 登录前(打开MsgShowWin面板前)获取账户借书信息
			 * 根据登录账号获取所现借书信息
			 */
			Vector<Vector<Object>> vectorUserBorrowMsg = new Vector<Vector<Object>>();
			dataAccount = checkAccount.transmitAccountMsg();
			String strAccountName = String.valueOf(dataAccount.get(0));
			String strSearchUserBorrowMsg = "SELECT * FROM borrowBookMsg WHERE borrowAccountName = " + "'" + strAccountName + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(strSearchUserBorrowMsg);
				while(resultSet.next()){
					Vector<Object> vectorOne = new Vector<Object>();
					for(int i = 2; i <= 5; i++){
						vectorOne.add(resultSet.getString(i));
					}
					vectorUserBorrowMsg.add(vectorOne);
				}
				resultSet.close();
				statement.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return vectorUserBorrowMsg;
		}
	
	//	public Vector<Object> getAccountMsg(String accountName){
	//		/*
	//		 * 登录后,用来刷新数据
	//		 * 通过登录名来获取账户信息
	//		 */
	//		Vector<Object> vectorAccountMsg = new Vector<Object>();
	//		String sqlSearchAccountMsg = "SELECT * FROM accountMsg WHERE accountName = " + "'" + accountName + "'";
	//		Connection connection = new ConnectionDatabase().conectionDatabse();
	//		try {
	//			Statement statement = connection.createStatement();
	//			ResultSet resultSet = statement.executeQuery(sqlSearchAccountMsg);
	//			while(resultSet.next()){
	//				for(int i = 1; i <= 6; i++){
	//					vectorAccountMsg.add(resultSet.getString(i));
	//				}
	//			}
	//			resultSet.close();
	//			statement.close();
	//		} catch (SQLException e) {
	//			// TODO Auto-generated catch block
	//			e.printStackTrace();
	//		}
	//		
	//		try {
	//			connection.close();
	//		} catch (SQLException e) {
	//			// TODO Auto-generated catch block
	//			e.printStackTrace();
	//		}
	//		
	//		return vectorAccountMsg;
	//	}
	
		public Vector<Vector<Object>> getUserBorrowMsg(String accountName){
			/*
			 * 登录后获取,用来刷新数据
			 * 根据登录账号名获取所现借书信息
			 */
			Vector<Vector<Object>> vectorUserBorrowMsg = new Vector<Vector<Object>>();
			String sqlSearchUserBorrowMsg = "SELECT * FROM borrowBookMsg WHERE borrowAccountName = " + "'" + accountName + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(sqlSearchUserBorrowMsg);
				while(resultSet.next()){
					Vector<Object> vectorOne = new Vector<Object>();
					for(int i = 2; i <= 5; i++){
						vectorOne.add(resultSet.getString(i));
					}
					vectorUserBorrowMsg.add(vectorOne);
				}
				resultSet.close();
				statement.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return vectorUserBorrowMsg;
		}
	
		public String searchBook(String strjtfInputIndexBorrow){
			/*
			 * 根据索引来查找图书名称,并返回
			 */
			String str = "未找到此书";
			String sqlSearchBook = "SELECT bookName FROM allBook WHERE bookIndex = " + "'" + strjtfInputIndexBorrow + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(sqlSearchBook);
			
				while(resultSet.next()){
					str = resultSet.getString(1);
				}
				resultSet.close();
				statement.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return str;
		}
	
		public String borrowBook(String strjtfInputIndexBorrow, String strjtfShowBookBorrow, String accountName){
			/*
			 * 借书过程,
			 * 先通过传递过来的书名,通过书名来在borrowBookMsg所有已经借出去的书的信息库中对比,确认该库中是否有这本书(若有,则表示该书已经借出去)
			 * 若有,继而再判断是不是现在的登录账户借了,如果不是,则返回已经被别人借走,
			 * 若无,则开始借书过程
			 */
			String str = null;
			String sqlCheckAccountWetherBorrow = "SELECT borrowAccountName FROM borrowBookMsg WHERE borrowBookName = " + "'" + strjtfShowBookBorrow + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(sqlCheckAccountWetherBorrow);
				Vector<Object> vectorAccountName = new Vector<Object>();
				while(resultSet.next()){
					vectorAccountName.add(resultSet.getString(1));
				}
				if(vectorAccountName.size() != 0){
					if(vectorAccountName.get(0).equals(accountName)){
						str = "你已借阅该图书,请注意保管";
					}else{
						str = "该图书已被他人借阅";
					}
				}else{
	//				System.out.println("该书在架上");
					SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy.MM.dd");
					String borrowTime = simpleDateFormat.format(new Date());
					String borrowNeedReturnTime = getNeedReturnTime();
					String sqlBorrowInsert = "INSERT INTO borrowBookMsg(borrowAccountName, borrowBookIndex, borrowBookName, borrowTime, borrowNeedReturnTime)"
							+ "VALUES" + "(" + "'" + accountName + "'" + "," + "'" + strjtfInputIndexBorrow + "'" + "," + "'" + strjtfShowBookBorrow + "'" + "," 
							+ "'" + borrowTime + "'" + "," + "'"+ borrowNeedReturnTime + "'" + ")";
					int n = statement.executeUpdate(sqlBorrowInsert);
					if(n > 0){
						str = "借书成功";
					}else{
						str = "借书失败";
					}
				}
				resultSet.close();
				statement.close();
			
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return str;
		}
	
		public String returnBook(String strjtfShowBookReturn){
			/*
			 * 还书
			 */
			String str = null;
			String sqlCheckWetherBorrow = "SELECT borrowAccountName FROM borrowBookMsg WHERE borrowBookName = " + "'" + strjtfShowBookReturn + "'";
			String sqlReturnBook = "DELETE FROM borrowBookMsg WHERE borrowBookName = " + "'" + strjtfShowBookReturn + "'";
			Connection connection = new ConnectionDatabase().conectionDatabse();
			try {
				Statement statement = connection.createStatement();
				ResultSet resultSet = statement.executeQuery(sqlCheckWetherBorrow);
				Vector<Object> vectorAccountName = new Vector<Object>();
				while(resultSet.next()){
					vectorAccountName.add(resultSet.getString(1));
				}
				if(vectorAccountName.size() != 0){
					int n = statement.executeUpdate(sqlReturnBook);
					if( n > 0){
						str = "还书成功";
					}else{
						str = "还书失败";
					}
				}else{
					str = "该书未借出";
				}
				resultSet.close();
				statement.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return str;
		}
	
		public String getNeedReturnTime(){
			/*
			 * 获取当前系统时间的下一个月时间,并返回
			 */
			String str = null;
			Calendar calendar = Calendar.getInstance();
			calendar.setTime(new Date());
			calendar.add(Calendar.MONTH, +1);
			Date dateNextMonth = calendar.getTime();
			SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy.MM.dd");
			str = simpleDateFormat.format(dateNextMonth);
			return str;
		}
	
		public String checkConnectDatabase(){
			/*
			 * for MsgWin窗口
			 */
			String str = null;
			Connection connection = new ConnectionDatabase().conectionDatabse();
			if(connection != null){
				str = "已连接数据库";
			}else{
				str = "未连接数据库";
			}
			try {
				connection.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return str;
		}
		
	}

其中的方法borrowBook()显示通过书名,来与已经借出去的书的信息库中进行对比,也就是有一张表专门用来存放记录所有已经借出去的书籍

这样一来,如果对比到了的话说明这本书已经借出去,然而借出去的话有两种情况,一种是被自己借了,另一种就是被别人借走了,所以如果表中有该书名,那么还要根据表来对比账号名

所以这样做,使得借书和还书都更简单了,所以借书的道理也差不多

当然,这里我也自定义了一个对话框,

如下对话框代码,文加名字为

#### MyJDialog.java:

	package com.cgtest.ui;

	import java.awt.BorderLayout;
	import java.awt.event.ActionEvent;
	import java.awt.event.ActionListener;

	import javax.swing.JButton;
	import javax.swing.JDialog;
	import javax.swing.JFrame;
	import javax.swing.JLabel;
	import javax.swing.JPanel;
	import javax.swing.SwingUtilities;
	import javax.swing.UIManager;
	import javax.swing.UnsupportedLookAndFeelException;


	/**
	 * @author cg
	 *自定义对话框，
	 *如果将组件属性设置为static，那么将会出现变量值不会初始化
	 *pack()方法和validate()方法不能共存
	 *showChoiceForMsgWin(),drawShowChoiceForMsgWin()这两个方法是专门为系统注销登录而写的消息窗口,代码臃肿.重复,,,,日后改进-----2016.11.13
	 */
	public class MyJDialog{
		public JFrame jframe;
		public JDialog jdialog;
		public JLabel labelShow;
		public JButton btEnter,btCancel;
		public JPanel panelCenter = new JPanel(),panelSouth = new JPanel();
		boolean modal;
	
		public int index = 0, indexChoice = 0;
	
		public MyJDialog(boolean modal, JFrame jframe){
			this.modal = modal;
			this.jframe = jframe;
		}
	
		public void showMessage(String str){
			SwingUtilities.invokeLater(new Runnable() {
			
				@Override
				public void run() {
					try {
						UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
					} catch (ClassNotFoundException | InstantiationException | IllegalAccessException
							| UnsupportedLookAndFeelException e) {
						e.printStackTrace();
					}
					drawShowMessage(str);
				}
			});
		
		}
		public void drawShowMessage(String str){
			jdialog = new JDialog();
			jdialog.setModal(modal);
		
			jdialog.add(panelCenter,BorderLayout.CENTER);		
			jdialog.add(panelSouth, BorderLayout.SOUTH);
		
			labelShow = new JLabel(str);
			panelCenter.add(labelShow);
		
			btEnter = new JButton("确定");
			panelSouth.add(btEnter);
			btEnter.addActionListener(new ActionListener() {
				@Override
				public void actionPerformed(ActionEvent e) {
						jdialog.dispose();
				}
			});
		
			jdialog.setTitle("消息");
			jdialog.pack();
			jdialog.setResizable(false);
			int widthDia = (jframe.getX()+(jframe.getWidth())/2) - jdialog.getWidth()/2;
			int heightDia = (jframe.getY() + (jframe.getHeight()/2) - jdialog.getHeight()/2);
			jdialog.setLocation(widthDia, heightDia);
			jdialog.setVisible(true);
		}
		public void showChoice(String str){
			SwingUtilities.invokeLater(new Runnable() {
			
				@Override
				public void run() {
					try {
						UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
					} catch (ClassNotFoundException | InstantiationException | IllegalAccessException
							| UnsupportedLookAndFeelException e) {
						e.printStackTrace();
					}
					drawShowChoice(str);
				}
			});
		}
		public void drawShowChoice(String str){
			jdialog = new JDialog();
			jdialog.setModal(modal);
		
			jdialog.add(panelCenter,BorderLayout.CENTER);
			jdialog.add(panelSouth, BorderLayout.SOUTH);
		
			labelShow = new JLabel(str);
			panelCenter.add(labelShow);
		
			btEnter = new JButton("确定");
			panelSouth.add(btEnter);
			btEnter.addActionListener(new ActionListener() {
			
				@Override
				public void actionPerformed(ActionEvent e) {
					jdialog.dispose();
					jframe.dispose();
					System.exit(0);
				}
			});
		
			btCancel = new JButton("取消");
			panelSouth.add(btCancel);
			btCancel.addActionListener(new ActionListener() {
			
				@Override
				public void actionPerformed(ActionEvent e) {
					jdialog.dispose();
				}
			});
		
			jdialog.setTitle("请注意");
			jdialog.pack();
			jdialog.setResizable(false);
			int widthDia = (jframe.getX()+(jframe.getWidth())/2) - jdialog.getWidth()/2;
			int heightDia = (jframe.getY() + (jframe.getHeight()/2) - jdialog.getHeight()/2);
			jdialog.setLocation(widthDia, heightDia);
			jdialog.setVisible(true);
		
		}
	
		public void showChoiceForMsgWin(String str){
			SwingUtilities.invokeLater(new Runnable() {
			
				@Override
				public void run() {
					try {
						UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
					} catch (ClassNotFoundException | InstantiationException | IllegalAccessException
							| UnsupportedLookAndFeelException e) {
						e.printStackTrace();
					}
					drawShowChoiceForMsgWin(str);
				}
			});
		}
		public void drawShowChoiceForMsgWin(String str){
			jdialog = new JDialog();
			jdialog.setModal(modal);
		
			jdialog.add(panelCenter,BorderLayout.CENTER);
			jdialog.add(panelSouth, BorderLayout.SOUTH);
		
			labelShow = new JLabel(str);
			panelCenter.add(labelShow);
		
			btEnter = new JButton("确定");
			panelSouth.add(btEnter);
			btEnter.addActionListener(new ActionListener() {
			
				@Override
				public void actionPerformed(ActionEvent e) {
					jdialog.dispose();
					jframe.dispose();
	//				System.exit(0);
					try {
						Thread.sleep(600);                     //神奇的代码,dispose不能关闭线程,system.exit(0)可以
						new LoginWin();
					} catch (InterruptedException e1) {
						// TODO Auto-generated catch block
					}
	//				System.out.println("启动登录窗口");
				}
			});
		
			btCancel = new JButton("取消");
			panelSouth.add(btCancel);
			btCancel.addActionListener(new ActionListener() {
			
				@Override
				public void actionPerformed(ActionEvent e) {
					jdialog.dispose();
				}
			});
		
			jdialog.setTitle("请注意");
			jdialog.pack();
			jdialog.setResizable(false);
			int widthDia = (jframe.getX()+(jframe.getWidth())/2) - jdialog.getWidth()/2;
			int heightDia = (jframe.getY() + (jframe.getHeight()/2) - jdialog.getHeight()/2);
			jdialog.setLocation(widthDia, heightDia);
			jdialog.setVisible(true);
		
		}
	//	public int choiceTrigger(String str){
	//		index = showChoice(str);
	//		return index;
	//	}

	}

上面的代码只是自定义对话框,代码有点臃肿,

然后就是main函数了

如下代码,文件名为

#### BookControlSystem.java:

	package com.cgtest.main;

	import com.cgtest.ui.LoginWin;

	/**
	 * @author cg
	 * 图书管理系统
	 *基于linux系统上的xampp软件包上开发运行
	 *实现用户登录,借书,还书
	 *,
	 */
	public class BookControlSystem {
		public static void main(String [] args){
			new LoginWin();
		}
	}

最后,贴下

#### 数据库的建立脚本:

	CREATE DATABASE bookControlSystem;
	USE bookControlSystem;
	CREATE TABLE accountMsg(accountName char(20) PRIMARY KEY,accountPasswd char(20),accountPersonName char(20),accountSex char(5),accountAge int,accountContactInfo char(20))
	INSERT INTO accountMsg(accountName,accountPasswd,accountPersonName,accountSex,
	accountAge,accountContactInfo) VALUES ('V201441122','201441122','王珉','男','20','18062449761');
	INSERT INTO accountMsg(accountName,accountPasswd,accountPersonName,accountSex,
	accountAge,accountContactInfo) VALUES ('V201441123','201441123','罗洋','男','21','1390721584');
	INSERT INTO accountMsg(accountName,accountPasswd,accountPersonName,accountSex,
	accountAge,accountContactInfo) VALUES ('V201441035','wuhan1035','王厕所','女','20','18062249061');


	CREATE TABLE borrowBookMsg(borrowAccountName char(20) ,borrowBookIndex char(20),borrowBookName char(20),borrowTime char(20),borrowNeedReturnTime char(20));
	INSERT INTO borrowBookMsg(borrowAccountName,borrowBookIndex,borrowBookName,
	borrowTime,borrowNeedReturnTime) VALUES ('V201441122',
	'TP221-21','檀香刑','2016.11.1','2016.12.1');
	INSERT INTO borrowBookMsg(borrowAccountName,borrowBookIndex,borrowBookName,
	borrowTime,borrowNeedReturnTime) VALUES ('V201441122',
	'TN231-34','Java编程思想','2016.11.11','2016.12.11');
	INSERT INTO borrowBookMsg
	(borrowAccountName,borrowBookIndex,borrowBookName,
	borrowTime,borrowNeedReturnTime) VALUES ('V201441123',
	'TB124-33','C语言入门','2016.11.05','2016.12.05');

	CREATE TABLE allBook(bookIndex char(20) PRIMARY KEY,bookName char(20));
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TP221-21','檀香刑');
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TN231-34','Java编程思想');
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TB124-33','C语言入门');
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TB124-30','C语言从入门到精通');
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TP221-20','丰乳肥臀');
	INSERT INTO allBook(bookIndex,bookName) VALUES ('TN231-33','Java语言入门');

至此,就完成了图书馆管理系统的建立

将代码整合在各个包中,并创建该文件,配置好数据库服务器地址,用户名和密码就可以运行了

代码有点臃肿,我到日后改进

谢谢

[驱动下载页面]:https://dev.mysql.com/downloads/connector/j/

