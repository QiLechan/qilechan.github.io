---
title: 基于Java的加密通讯软件
tags:
- 技术
categories:
- 技术
date: 2023-03-18
---
## 一、引言
随着互联网的普及和发展，人们越来越需要一种方便快捷、安全可靠的即时通讯工具。传统的即时通讯软件大多基于中心化架构，即需要依赖一个中心服务器进行消息的转发和管理，这种架构存在单点故障、信息泄漏等问题。因此，越来越多的人开始关注去中心化的端对端通讯技术，即直接将消息传递给目标用户，避免了中心服务器的干扰和风险。

由于技术原因，我们无法实现真正的端对端通讯。但可以实现快速部署的加密通信服务端用于替代。虽然本质上仍然属于中心化架构，但服务端可以由安装者自由控制，所有数据都经过非对称加密的方式加密传输，除非转发服务器被攻击导致数据泄露，基本不存在信息泄露的可能性。

## 二、背景
为了满足这一需求，我们设计并实现了一款基于Java的加密通讯软件，使用GitHub进行版本控制和协作开发。服务端通过生成密钥对，用户可以通过安全的方式将服务端公钥部署到客户端，客户端每次发信使用服务端公钥进行RSA加密，服务端接收到加密信息后使用私钥解密，然后使用另一客户端发送到服务端的客户端公钥进行加密，再传输至另一客户端，客户端使用私钥进行解密。在每一次客户端接收加密信息之前，客户端都会重新生成一对密钥对并将公钥发送至服务端，提高了通讯的安全性。
同时，该软件具有良好的跨平台性能，可在不同操作系统和设备上运行，提高了用户的使用体验和便捷性。

## 三、目的
本研究的主要目的是探究如何利用Java语言开发一款加密通讯软件，并基于GitHub等工具实现版本控制、协作开发和自动构建。具体目标如下：
1.分析非对称加密通讯技术的原理和特点，研究如何使用Java语言实现加密通讯功能。
2.设计并实现一款基于Java的加密通讯软件，实现用户注册、登录、发送消息等基本功能。
3.使用GitHub进行版本控制、协作开发和自动构建，协同开发团队进行代码的修改和更新。
4.评估该软件的性能、安全性和用户体验等方面，并讨论其优缺点和未来改进方向。

## 三、方法
为了实现上述目标，我们采用了如下方法：
1.学习端对端通讯技术原理和相关技术，研究如何使用Java语言实现端对端通讯功能。
2.设计并实现一款基于Java的加密通讯软件，采用RSA加密算法，支持用户注册、登录、发送消息等基本功能。
3. 使用GitHub进行版本控制和协作开发，建立仓库并维护代码，实现团队协作和代码的修改和更新。
4.对软件进行性能、安全性和用户体验等方面的评估和测试，并针对评估结果进行分析和讨论，探讨未来的改进方向。

## 四、实现
在本研究中，我们使用Java语言实现了一款基于RSA算法的加密通讯软件。该软件的功能包括用户注册、登录、发送消息等基本功能，同时具有跨平台、易用、安全可靠的特点。以下是该软件的实现过程和具体功能：
1.确定开发框架：我们使用Java语言作为主要开发语言，以命令行应用程序作为界面，使用Socket和ServerSocket实现通讯功能。
2.用户注册和登录：用户注册和登录功能是软件的核心功能之一，用户需要在启动程序后输入用户名和密码进行登录。若用户未注册，则需先进行注册，注册时需要输入用户名、密码等信息，完成后即可登录。
3.发送消息：用户可以向服务器发送消息，该消息将经过服务器广播给所有连接至服务器的用户，没有服务端公钥的客户端不会被接受。消息发送后，接收方将收到通知并可以查看消息。

项目的源码位于https://github.com/QiLechan/JavaIM ，仓库遵循GPL-3.0开源协议，用户有运行、复制软件的自由，发行传播软件的自由，获得软件源码的自由，改进软件并将自己作出的改进版本向社会发行传播的自由，作者不承担使用者的任何责任。
项目使用Maven进行构建。该报告完成日期为2023年3月31日，最后一次提交的commit号为4910b55，内容可能与实际程序不符合。

### （一）服务端功能实现
我们首先实现的是服务端的功能。主要选择使用了java.net.ServerSocket和java.net.Socket库来实现服务端与客户端之间的握手与传输信息，org.apache.logging.log4j库用于输出信息，java.io.File库用于公钥文件的输出。
Server.java的主类：
```java
    public Server(int port) {
        instance = this;
        RSA_KeyAutogenerate();
        try {
            serverSocket = new ServerSocket(port);
            /* serverSocket.setSoTimeout(10000); */
        } catch (IOException e) {
            SaveStackTrace.saveStackTrace(e);
        }
        Runnable SQLUpdateThread = () -> {
            try {
                Connection mySQLConnection = Database.Init(CodeDynamicConfig.GetMySQLDataBaseHost(), CodeDynamicConfig.GetMySQLDataBasePort(), CodeDynamicConfig.GetMySQLDataBaseName(), CodeDynamicConfig.GetMySQLDataBaseUser(), CodeDynamicConfig.GetMySQLDataBasePasswd());
                String sql = "UPDATE UserData SET UserLogged = 0 where UserLogged = 1;";
                PreparedStatement ps = mySQLConnection.prepareStatement(sql);
                ps.executeUpdate();
            }
            catch (Exception e)
            {
                SaveStackTrace.saveStackTrace(e);
            }
        };
        Thread UpdateThread = new Thread(SQLUpdateThread);
        UpdateThread.start();
        UpdateThread.setName("SQL Update Thread");
        try {
            UpdateThread.join();
        } catch (InterruptedException e) {
            logger.error("发生异常InterruptedException");
            SaveStackTrace.saveStackTrace(e);
        StartUserAuthThread();
        StartTimer();
        //这里"getInstance"其实并不是真的为了获取实例，而是因为启动PluginManager在构造函数
        PluginManager.getInstance("./plugins");
        StartCommandSystem();
    }
}
```
https://github.com/QiLechan/JavaIM/blob/main/src/main/java/org/yuezhikong/Server/Server.java

StartUserAuthThread()是用于实现用户登录的方法。方法首先创建一个新线程UserAuthThread，使用一个无限循环不断等待用户连接，并处理连接请求。
StartTimer()用于启动一个定时器用于计时。
StartCommandSystem()使用一个无限循环不断接收用户的输入的指令。
当服务端开始运行时，会提示输入要监听的端口号，然后根据配置文件（但是是在源代码中）中的变量决定服务端可以接受连接的总数量。检测目录下是否存在公钥，如果不存在，则重新生成新的公钥。如果检测有私钥但没有公钥，或存在公钥没有私钥，则会重新覆盖新的密钥对。
完成密钥生成后，开始处理用户登录的过程，登录的用户会被加入到Users的List中，之后启动接收消息线程，服务器开始消息广播。
用于实现对信息解密和重新加密发送的类位于https://github.com/QiLechan/JavaIM/blob/15ad975ddd3268e4f4ddcc93b9d680d776377b95/src/main/java/org/yuezhikong/Server/UserData/RecvMessageThread.java ，它主要通过调用接口ServerAPI中的SendMessageToAllClient()函数实现发信功能，以及RSA类进行加解密和输出密钥对，SendMessageToAllClient()内容如下：
![code2](https://blog.qileoffice.top/images/carbon.png)
在RSA.java中，最主要使用了第三方库hutool更加方便地实现了RSA加密的过程：
![code3](https://blog.qileoffice.top/images/carbon1.png)
另外为了加强安全性，要进行RSA加密的信息首先会进行加盐。

### （二）客户端功能实现
客户端则与服务端相似，在运行前首先检查是否在运行目录下有服务端密钥，如果没有将会停止运行。客户端使用无限循环获取用户输入的信息。客户端开始运行时，会要求输入服务器的IP以及端口号，在输入正确的IP和端口后，将开始通讯握手。在握手成功后，服务端会向客户端发送响应信息，之后客户端开始等待用户输入信息。用户输入信息回车后，调用RSA类中的加密方法加密后发送至服务端。

### （三）主类
为了方便，我们将客户端和服务端在一个程序中实现，以下是程序主类Main.java：
```java
public static void main(String[] args) {
    try {
        if (GetAutoSaveDependencyMode()) {
            getInstance().saveLibFiles();
        }
        if (isThisVersionIsExpVersion())
        {
            logger.info("此版本为实验性版本！不会保证稳定性");
            logger.info("本版本存在一些正在开发中的内容，可能存在一些问题");
            logger.info("本版本测试性内容列表：");
            logger.info(getExpVersionText());
        }
        logger.info("欢迎来到JavaIM！版本："+getVersion());
        logger.info("使用客户端模式请输入1，服务端模式请输入2:");
        Scanner sc = new Scanner(System.in);
        int mode = sc.nextInt();
        if (mode == 1) {
            Scanner sc2 = new Scanner(System.in);
            Scanner sc3 = new Scanner(System.in);
            String serverName;
            logger.info("请输入要连接的主机:");
            serverName = sc2.nextLine();
            logger.info("请输入端口:");
            int port = Integer.parseInt(sc3.nextLine());
            new Client(serverName, port);
        } else if (mode == 2) {
            Scanner sc4 = new Scanner(System.in);
            logger.info("请输入监听端口:");
            int port = Integer.parseInt(sc4.nextLine());
            new Server(port);
        } else {
            logger.info("输入值错误，请重新运行程序");
        }
    }
    catch (Exception e)
    {
        SaveStackTrace.saveStackTrace(e);
    }
}
```

在程序开始运行时，会询问用户使用客户端模式还是服务端模式，并根据用户的选择运行。

### （四）用户管理
为实现用户管理，我们实现了MySQL数据库管理系统的接入和用户数据类，在user.java中，为用户定义了PermissionLevel（权限等级），UserName（用户名），UserLogined（用户登录状态），UserSocket（用户Socket连接），ClientID（用户客户端编号），UserPublicKey（用户公钥），PublicKeyChanged（公钥状态），MuteTime（禁言时间），Muted（是否禁言）等私有变量，并提供setMuted()，setMuteTime()，isMuted()等等用于对以上变量操作的方法供其他类调用。
以下是对数据库进行操作的类Database.java：
```java
public class Database {
    /**
     * 初始化数据库连接
     * @param host 如果为MySQL连接，那么为MySQL地址
     * @param port 如果为MySQL连接，那么为MySQL端口
     * @param Database 如果为MySQL连接，那么为MySQL数据库名称
     * @param UserName 如果为MySQL连接，那么为MySQL用户名
     * @param Password 如果为MySQL连接，那么为MySQL密码
     * @throws ClassNotFoundException 找不到数据库驱动
     * @throws SQLException 获取数据库连接时连接失败
     * @return 数据库连接
     */
    public static Connection Init(String host, String port, String Database, String UserName, String Password) throws ClassNotFoundException, SQLException {
        if (!CodeDynamicConfig.GetSQLITEMode()) {
            Class.forName("com.mysql.cj.jdbc.Driver");
            Connection DatabaseConnection = DriverManager.getConnection("jdbc:mysql://" + host + ":" + port + "/" + Database + "?autoReconnect=true&failOverReadOnly=false&maxReconnects=1000&serverTimezone=Asia/Shanghai&initialTimeout=1&useSSL=false", UserName, Password);
            String sql =
                    "CREATE TABLE if not exists UserData" +
                            " (" +
                            " DatabaseProtocolVersion INT,"+//Database Table协议版本
                            " UserMuted INT," +//是否已被禁言
                            " UserMuteTime BIGINT," +//用户禁言时长
                            " Permission INT," +//权限等级，目前只有三个等级，-1级：被封禁用户，0级：普通用户，1级：管理员
                            " UserName varchar(255)," +//用户名
                            " Passwd varchar(255)," +//密码
                            " salt varchar(255)," +//密码加盐加的盐
                            " UserLogged INT"+//用户是否已登录
                            " );";
            PreparedStatement ps = DatabaseConnection.prepareStatement(sql);
            ps.executeUpdate();
            sql = "select * from UserData where DatabaseProtocolVersion = 1";
            ps = DatabaseConnection.prepareStatement(sql);
            ResultSet rs = ps.executeQuery();
            while (rs.next())
            {
                sql = "INSERT  INTO `UserData` (" +
                        "`DatabaseProtocolVersion`" +
                        ",`UserMuted`," +
                        " `UserMuteTime`," +
                        "`Permission`," +
                        " `UserName`," +
                        "`Passwd`," +
                        "`salt`," +
                        "`UserLogged`" +
                        ") VALUES (?,?, ?, ?,?,?, ?,?);";
                ps = DatabaseConnection.prepareStatement(sql);
                ps.setInt(1, CodeDynamicConfig.GetDatabaseProtocolVersion());
                ps.setLong(2,rs.getLong("UserMuted"));
                ps.setLong(3,rs.getLong("UserMuteTime"));
                ps.setLong(4,rs.getLong("Permission"));
                ps.setString(5,rs.getString("UserName"));
                ps.setString(6,rs.getString("Passwd"));
                ps.setString(7,rs.getString("salt"));
                ps.setInt(8,0);
                ps.executeUpdate();
            }
            return DatabaseConnection;
        }
        else {
            Class.forName("org.sqlite.JDBC");
            Connection DatabaseConnection = DriverManager.getConnection("jdbc:sqlite:data.db");
            String sql =
                    "CREATE TABLE if not exists UserData" +
                            " (" +
                            " DatabaseProtocolVersion INT,"+//Database Table协议版本
                            " UserMuted INT," +//是否已被禁言
                            " UserMuteTime BIGINT," +//用户禁言时长
                            " Permission INT," +//权限等级，目前只有三个等级，-1级：被封禁用户，0级：普通用户，1级：管理员
                            " UserName varchar(255)," +//用户名
                            " Passwd varchar(255)," +//密码
                            " salt varchar(255)," +//密码加盐加的盐
                            " UserLogged INT"+//用户是否已登录
                            " );";
            PreparedStatement ps = DatabaseConnection.prepareStatement(sql);
            ps.executeUpdate();
            sql = "select * from UserData where DatabaseProtocolVersion = 1";
            ps = DatabaseConnection.prepareStatement(sql);
            ResultSet rs = ps.executeQuery();
            while (rs.next())
            {
                sql = "INSERT  INTO `UserData` (" +
                        "`DatabaseProtocolVersion`" +
                        ",`UserMuted`," +
                        " `UserMuteTime`," +
                        "`Permission`," +
                        " `UserName`," +
                        "`Passwd`," +
                        "`salt`," +
                        "`UserLogged`" +
                        ") VALUES (?,?, ?, ?,?,?, ?,?);";
                ps = DatabaseConnection.prepareStatement(sql);
                ps.setInt(1, CodeDynamicConfig.GetDatabaseProtocolVersion());
                ps.setLong(2,rs.getLong("UserMuted"));
                ps.setLong(3,rs.getLong("UserMuteTime"));
                ps.setLong(4,rs.getLong("Permission"));
                ps.setString(5,rs.getString("UserName"));
                ps.setString(6,rs.getString("Passwd"));
                ps.setString(7,rs.getString("salt"));
                ps.setInt(8,0);
                ps.executeUpdate();
            }
            return DatabaseConnection;
        }
    }
}
```

以下为处理用户登录的类UserLogin.java：
```java
public class UserLogin{
    /**
     * 是否允许用户登录
     * @param LoginUser 请求登录的用户
     * @return 是/否允许
     * @throws UserAlreadyLoggedInException 用户已经登录了
     * @throws NullPointerException 用户的某些信息读取出NULL
     * @apiNote 虽然在执行的期间，就会写入到user.class中，但也请您根据返回值做是否踢出登录等的处理
     */
    public static boolean WhetherTheUserIsAllowedToLogin(user LoginUser) throws UserAlreadyLoggedInException,NullPointerException {
        if (LoginUser.GetUserLogined())
        {
            throw new UserAlreadyLoggedInException("This User Is Logined!");
        }
        else
        {

            try {
                SendMessageToUser(LoginUser,"在进入之前，您必须先登录/注册");
                Thread.sleep(250);
                SendMessageToUser(LoginUser,"输入1进行登录");
                Thread.sleep(250);
                SendMessageToUser(LoginUser,"输入2进行注册");
                String UserSelect;
                BufferedReader reader = new BufferedReader(new InputStreamReader(LoginUser.GetUserSocket().getInputStream()));//获取输入流
                UserSelect = reader.readLine();
                if (UserSelect == null)
                {
                    throw new NullPointerException();
                }
                if (GetRSA_Mode()) {
                    if (isAES_Mode())
                    {
                        UserSelect = LoginUser.GetUserAES().decryptStr(UserSelect);
                    }
                    else
                        UserSelect = RSA.decrypt(UserSelect,Objects.requireNonNull(RSA.loadPrivateKeyFromFile("Private.key")).PrivateKey);
                }
                UserSelect = java.net.URLDecoder.decode(UserSelect, StandardCharsets.UTF_8);
                int Select = Integer.parseInt(UserSelect);
                SendMessageToUser(LoginUser,"请输入您的用户名");
                String UserName;
                reader = new BufferedReader(new InputStreamReader(LoginUser.GetUserSocket().getInputStream()));//获取输入流
                UserName = reader.readLine();
                if (UserName == null)
                {
                    throw new NullPointerException();
                }
                if (GetRSA_Mode()) {
                    if (isAES_Mode())
                    {
                        UserName = LoginUser.GetUserAES().decryptStr(UserName);
                    }
                    else
                        UserName = RSA.decrypt(UserName,Objects.requireNonNull(RSA.loadPrivateKeyFromFile("Private.key")).PrivateKey);
                }
                UserName = java.net.URLDecoder.decode(UserName, StandardCharsets.UTF_8);
                SendMessageToUser(LoginUser,"请输入您的密码");
                String Password;
                reader = new BufferedReader(new InputStreamReader(LoginUser.GetUserSocket().getInputStream()));//获取输入流
                Password = reader.readLine();
                if (Password == null)
                {
                    throw new NullPointerException();
                }
                if (GetRSA_Mode()) {
                    if (isAES_Mode())
                    {
                        Password = LoginUser.GetUserAES().decryptStr(Password);
                    }
                    else {
                        Password = RSA.decrypt(Password, Objects.requireNonNull(RSA.loadPrivateKeyFromFile("Private.key")).PrivateKey);
                    }
                }
                Password = java.net.URLDecoder.decode(Password, StandardCharsets.UTF_8);
                //上方为请求用户输入用户名、密码
                boolean ThisUserNameIsNotLogin = false;
                try {
                    ServerAPI.GetUserByUserName(UserName, Server.GetInstance());
                } catch (UserNotFoundException e)
                {
                    ThisUserNameIsNotLogin = true;
                }
                if (!ThisUserNameIsNotLogin)
                {
                    throw new UserAlreadyLoggedInException("This User Is Logined!");
                }
                //上方为处理此用户是否已登录
                if (Select == 1)//登录
                {
                    //下方为SQL处理
                    UserLoginRequestThread loginRequestThread = new UserLoginRequestThread(LoginUser,UserName,Password);
                    loginRequestThread.start();
                    loginRequestThread.setName("UserLoginRequest");
                    loginRequestThread.join();
                    if (!loginRequestThread.GetReturn())
                    {
                        SendMessageToUser(LoginUser,"抱歉，您的本次登录被拒绝");
                    }
                    else
                    {
                        SendMessageToUser(LoginUser,"登录成功！");
                    }
                    return loginRequestThread.GetReturn();
                }
                else if (Select == 2)//注册
                {
                    //下方为SQL处理
                    UserRegisterRequestThread RegisterRequestThread = new UserRegisterRequestThread(LoginUser,UserName,Password);
                    RegisterRequestThread.start();
                    RegisterRequestThread.setName("UserRegisterThread");
                    RegisterRequestThread.join();
                    if (!RegisterRequestThread.GetReturn())
                    {
                        SendMessageToUser(LoginUser,"抱歉，您的本次注册被拒绝");
                    }
                    else
                    {
                        SendMessageToUser(LoginUser,"注册成功！");
                    }
                    return RegisterRequestThread.GetReturn();
                }
                else
                {
                    SendMessageToUser(LoginUser,"非法输入！这里只允许输入1/2");
                    SendMessageToUser(LoginUser,"由于您不遵守提示，系统将终止您的会话！");
                    return false;
                }
            }
            catch (IOException e)
            {
                org.yuezhikong.utils.SaveStackTrace.saveStackTrace(e);
            }
            catch (NumberFormatException e)
            {
                SendMessageToUser(LoginUser,"非法输入！这里只允许输入1/2");
                SendMessageToUser(LoginUser,"由于您不遵守提示，系统将终止您的会话！");
            } catch (InterruptedException e) {
                SendMessageToUser(LoginUser,"出现内部异常，无法完成此操作");
                return false;
            }
            return false;
        }
    }
}
```

以下为服务端处理用户登录请求的类UserLoginRequestThread：
```java
public class UserLoginRequestThread extends Thread{
    private boolean RequestReturn = false;
    private final user RequestUser;
    private String Username;
    private final String Passwd;
    public UserLoginRequestThread(user LoginUser,String UserName,String Password)
    {
        RequestUser = LoginUser;
        Username = UserName;
        Passwd = Password;
    }
    @Override
    public void run() {
        super.run();
        try {
            Connection mySQLConnection = Database.Init(CodeDynamicConfig.GetMySQLDataBaseHost(), CodeDynamicConfig.GetMySQLDataBasePort(), CodeDynamicConfig.GetMySQLDataBaseName(), CodeDynamicConfig.GetMySQLDataBaseUser(), CodeDynamicConfig.GetMySQLDataBasePasswd());
            String sql = "select * from UserData where UserName = ?";
            PreparedStatement ps = mySQLConnection.prepareStatement(sql);
            ps.setString(1,Username);
            ResultSet rs = ps.executeQuery();
            String salt;
            String sha256;
            while (rs.next())
            {
                if (rs.getInt("UserLogged") == 1)
                {
                    break;
                }
                salt = rs.getString("salt");
                sha256 = SecureUtil.sha256(Passwd + salt);
                if (rs.getString("Passwd").equals(sha256))
                {
                    Username = rs.getString("UserName");
                    int PermissionLevel = rs.getInt("Permission");
                    if (PermissionLevel != 0)
                    {
                        if (PermissionLevel != 1)
                        {
                            if (PermissionLevel != -1)
                            {
                                PermissionLevel = 0;
                            }
                            else
                            {
                                SendMessageToUser(RequestUser,"您的账户已被永久封禁！");
                                RequestReturn = false;
                                mySQLConnection.close();
                                return;
                            }
                        }
                    }
                    long muted = rs.getLong("UserMuted");
                    long MuteTime = rs.getLong("UserMuteTime");
                    if (muted == 1)
                    {
                        RequestUser.setMuteTime(MuteTime);
                        RequestUser.setMuted(true);
                    }
                    RequestReturn = true;
                    RequestUser.SetUserPermission(PermissionLevel,true);
                    RequestUser.UserLogin(Username);
                    sql = "UPDATE UserData SET UserLogged = 1 where UserName = ?;";
                    ps = mySQLConnection.prepareStatement(sql);
                    ps.setString(1,Username);
                    ps.executeUpdate();
                    mySQLConnection.close();
                    return;
                }
            }
            mySQLConnection.close();
            RequestReturn = false;
        }
        catch (ClassNotFoundException e)
        {
            StringWriter sw = new StringWriter();
            PrintWriter pw = new PrintWriter(sw);
            e.printStackTrace(pw);
            pw.flush();
            sw.flush();
            logger_log4j.debug(sw.toString());
            pw.close();
            try {
                sw.close();
            }
            catch (IOException ex)
            {
                ex.printStackTrace();
            }
            logger_log4j.fatal("ClassNotFoundException，无法找到MySQL驱动");
            logger_log4j.fatal("程序已崩溃");
            System.exit(-2);
            RequestReturn = false;
        }
        catch (SQLException e)
        {
            org.yuezhikong.utils.SaveStackTrace.saveStackTrace(e);
            RequestReturn = false;
        }
    }

    public boolean GetReturn() {
        return RequestReturn;
    }
}
```

以下为处理用户注册的类UserRegisterRequestThread.java：
```java
public class UserRegisterRequestThread extends Thread{
    private boolean RequestReturn;
    private final org.yuezhikong.Server.UserData.user user;
    private final String Username;
    private final String Passwd;
    public boolean GetReturn() {
        return RequestReturn;
    }

    public UserRegisterRequestThread(user RequestUser,String username,String Password)
    {
        user = RequestUser;
        Username = username;
        Passwd = Password;
    }
    @Override
    public void run() {
        super.run();
        String salt = UUID.randomUUID().toString();
        String sha256 = SecureUtil.sha256(Passwd + salt);
        try {
            Connection DatabaseConnection = Database.Init(CodeDynamicConfig.GetMySQLDataBaseHost(), CodeDynamicConfig.GetMySQLDataBasePort(), CodeDynamicConfig.GetMySQLDataBaseName(), CodeDynamicConfig.GetMySQLDataBaseUser(), CodeDynamicConfig.GetMySQLDataBasePasswd());
            String sql = "select * from UserData where UserName = ?";
            PreparedStatement ps = DatabaseConnection.prepareStatement(sql);
            ps.setString(1,Username);
            ResultSet rs  = ps.executeQuery();
            if (rs.next())
            {
                RequestReturn = false;
                DatabaseConnection.close();
                return;
            }
            sql = "INSERT INTO `UserData` (`Permission`,`UserName`, `Passwd`,`salt`) VALUES (0,?, ?, ?);";
            ps = DatabaseConnection.prepareStatement(sql);
            ps.setString(1,Username);
            ps.setString(2,sha256);
            ps.setString(3,salt);
            ps.executeUpdate();
            DatabaseConnection.close();
            RequestReturn = true;
            user.UserLogin(Username);
        } catch (ClassNotFoundException | SQLException e) {
            user.UserDisconnect();
        }

    }
}
```

### （五）插件系统（Beta）
在进一步的开发过程中，我们实现了Java的程序的动态加载系统。通过我们封装的接口，开发者可以方便地进行JavaIM插件的开发。
目前已经实现服务端的接口有SendMessageToUsers()，SendMessageToAllClient()，GetUserByUsersName()。插件的开发示例在仓库的ExamplePlugin分支：https://github.com/QiLechan/JavaIM/tree/ExamplePlugin
该系统仍然处于实验性阶段。
以下是插件管理器PluginManager.java：
```java
public class PluginManager {
    private final List<Plugin> PluginList = new ArrayList<>();
    private int NumberOfPlugins = 0;
    private static PluginManager Instance;

    public static PluginManager getInstance(String DirName) {
        if (CodeDynamicConfig.GetPluginSystemMode())
        {
            if (Instance == null)
            {
                Instance = new PluginManager(DirName);
            }
            return Instance;
        }
        else {
            return null;
        }
    }

    /**
     * 获取文件夹下所有.jar结尾的文件
     * 一般是插件文件
     * @param DirName 文件夹路径
     * @return 文件列表
     */
    private List<File> GetPluginFileList(String DirName)
    {
        File file = new File(DirName);
        if (file.isDirectory())
        {
            String[] list = file.list();
            if (list == null)
            {
                return null;
            }
            List<File> PluginList = new ArrayList<>();
            for (String s : list) {
                if (s.toLowerCase(Locale.ROOT).endsWith(".jar")) {
                    File file1 = new File(DirName + "\\" + s);
                    PluginList.add(file1);
                }
            }
            return PluginList;
        }
        else
            return null;
    }

    /**
     * 加载一个文件夹下的所有插件
     * @param DirName 文件夹路径
     */
    public PluginManager(String DirName)
    {
        if (!(new File(DirName).exists()))
        {
            try {
                if (!(new File(DirName).mkdir())) {
                    org.yuezhikong.utils.Logger logger = new org.yuezhikong.utils.Logger();
                    logger.error("无法新建文件夹" + DirName + "，可能是由于权限问题");
                }
            }
            catch (Exception e)
            {
                org.yuezhikong.utils.Logger logger = new org.yuezhikong.utils.Logger();
                logger.error("无法新建文件夹"+DirName+"，可能是由于权限问题");
                org.yuezhikong.utils.SaveStackTrace.saveStackTrace(e);
            }
        }
        List<File> PluginFileList = GetPluginFileList(DirName);
        if (PluginFileList == null)
        {
            return;
        }
        for (File s : PluginFileList) {
            try {
                PluginJavaLoader classLoader = new PluginJavaLoader(ClassLoader.getSystemClassLoader(),s);
                classLoader.ThisPlugin.OnLoad(Server.GetInstance());
                PluginList.add(classLoader.ThisPlugin);
                NumberOfPlugins = NumberOfPlugins + 1;
            }
            catch (Throwable e)
            {
                    org.yuezhikong.utils.Logger logger = new org.yuezhikong.utils.Logger();
                    logger.error("加载插件文件"+s.getName()+"失败！请检查此插件！");
                    logger.error("发生此错误可能的原因是");
                    logger.error("1：插件内没有清单文件PluginManifest.properties");
                    logger.error("2：输入流InputStream异常，一般如果是此原因，是您文件权限导致的");
                    logger.error("3：插件内清单文件注册的主类无效");
                    logger.error("4：未根据插件要求进行继承");
                    logger.error("具体原因请看logs/debug.log内内容");
                    logger.error("如果下方报错原因为：");
                    logger.error("ClassNotFoundException则代表主类无效");
                    logger.error("IOException代表输入流InputStream异常");
                    logger.error("ClassCastException代表未根据要求继承");
                    logger.error("NullPointerException代表无清单文件或输入流InputStream无效");
                    logger.error("其他错误代表出现错误，请联系开发者排查");
                    logger.error("当前原因为："+e.getClass().getName()+" "+e.getMessage());
                    logger.error("请自行分辨原因");
                    SaveStackTrace.saveStackTrace(e);
                }
        }
    }

    /**
     * 慎用！
     * 执行此方法，会导致程序退出！请保证他在程序的退出流程最后
     * @param ProgramExitCode 程序退出时的代码
     */
    public void OnProgramExit(int ProgramExitCode)
    {
        if (NumberOfPlugins == 0)
        {
            System.exit(ProgramExitCode);
        }
        for (Plugin plugin : PluginList) {
            plugin.OnUnLoad(Server.GetInstance());
        }
        System.exit(ProgramExitCode);
    }

    /**
     * 用于调用插件事件处理程序
     * @param ChatUser 用户信息
     * @param Message 消息
     * @return true为阻止消息，false为正常操作
     */
    public boolean OnUserChat(user ChatUser,String Message)
    {
        boolean Block = false;
        if (NumberOfPlugins == 0)
        {
            return false;
        }
        for (Plugin plugin : PluginList) {
            if (plugin.OnChat(ChatUser,Message,Server.GetInstance()))
            {
                Block = true;
            }
        }
        return Block;
    }

    /**
     * 解除禁言时调用
     * @param UnMuteUser 被解除禁言的用户
     * @return 是否取消
     */
    public boolean OnUserUnMute(user UnMuteUser)
    {
        boolean Block = false;
        if (NumberOfPlugins == 0)
        {
            return false;
        }
        for (Plugin plugin : PluginList) {
            if (plugin.OnUserUnMuted(UnMuteUser,Server.GetInstance()))
            {
                Block = true;
            }
        }
        return Block;
    }

    /**
     * 发生权限更改时调用
     * @param PermissionChangeUser 被修改权限的用户
     * @return 是否取消
     */
    public boolean OnUserPermissionChange(user PermissionChangeUser,int NewPermissionLevel)
    {
        boolean Block = false;
        if (NumberOfPlugins == 0)
        {
            return false;
        }
        for (Plugin plugin : PluginList) {
            if (plugin.OnUserPermissionEdit(PermissionChangeUser,NewPermissionLevel,Server.GetInstance()))
            {
                Block = true;
            }
        }
        return Block;
    }

    /**
     * 禁言时调用
     * @param MuteUser 被禁言的用户
     * @return 是否取消
     */
    public boolean OnUserMute(user MuteUser)
    {
        boolean Block = false;
        if (NumberOfPlugins == 0)
        {
            return false;
        }
        for (Plugin plugin : PluginList) {
            if (plugin.OnUserMuted(MuteUser,Server.GetInstance()))
            {
                Block = true;
            }
        }
        return Block;
    }
}
```

### （六）安卓实现（开发中）
以上内容均为仅适用于PC端设备，作为即时通讯软件，不能没有便于携带的版本。为此，我们将代码移植到了安卓平台上，实现了在安卓手机上使用。
安卓版需要Android4.4以上的系统才可以运行。
安卓项目仍处于实验性阶段。
项目源码位于https://github.com/QiLechan/JavaIMForAndriod

## 五、应用实例
（略）

## 六、结果
在本研究中，我们成功地实现了一款基于Java的加密通讯软件，该软件具有跨平台、安全可靠等优点，并采用了GitHub进行版本控制和协作开发。以下是该软件的测试结果和评估：
（一）性能测试：我们对软件的性能进行了测试，测试包括登录和发送消息等功能的响应时间和稳定性等方面。测试结果表明，软件的响应速度较快，且稳定性较好。
（二）安全性测试：我们对软件的安全性进行了测试，测试包括数据加密和防止数据泄露等方面。测试结果表明，软件的数据传输采用了加密协议，防止了数据泄露和劫持等安全问题。
（三）用户体验测试：我们邀请了多位用户测试软件，他们对软件的功能实现、使用体验等方面都给予了较高的评价，认为该软件易于操作、功能齐全等。

## 七、讨论和改进
在本研究中，我们对开发基于Java的端对端通讯软件进行了探索和实现，并取得了一定的成果。但是，在软件开发和测试过程中，我们也发现了一些问题和不足之处，需要进一步改进和优化。以下是我们的讨论和改进方向：
（一）优化通讯功能：在实现加密通讯功能时，我们采用了Socket和ServerSocket进行通讯。但是，该方法存在连接数限制和安全问题等，需要进一步改进和优化。
（二）添加GUI界面：我们没有为软件实现GUI界面，因此，我们需要进一步实现界面设计，提高用户体验。
（三）完善功能和性能：虽然我们实现了基本的用户注册、登录和发送消息等功能，但是还有一些功能需要完善，如用户个人信息管理等。此外，我们也需要进一步优化软件的性能，提高响应速度和稳定性。
（四）探索新的开发框架和技术：随着技术的不断发展，新的开发框架和技术不断涌现。因此，我们也需要不断探索新的开发框架和技术，以便进一步优化和改进软件。

## 八、结论
本研究成功地实现了一款基于Java的端对端通讯软件，该软件具有跨平台、安全可靠等优点，并采用了GitHub进行版本控制、协作开发以及自动构建。在软件开发和测试过程中，我们也发现了一些问题和不足之处，需要进一步改进和优化。因此，我们将继续改进软件的功能和性能，并探索新的开发框架和技术，以便进一步提高软件的质量和用户体验。
总之，本研究对于基于Java的加密通讯软件的开发和实现进行了深入的探索和研究，并取得了一定的成果。通过对软件开发、测试、优化等方面的不断努力和探索，我们成功地实现了一款具有一定功能和优势的端对端通讯软件。此外，通过使用GitHub进行版本控制、协作开发和自动构建，也进一步提高了软件的开发效率和质量。
当然，本研究还存在一些不足和问题，需要进一步改进和完善。例如，在加密通讯功能、界面设计、功能完善和性能优化等方面，我们都需要进一步努力和探索。同时，随着技术的不断发展和创新，我们也需要不断探索和引入新的开发框架和技术，以便进一步提高软件的质量和用户体验。
最后，我们希望本研究能够对Java开发者和相关从业人员提供一些参考和帮助，同时也期待更多的研究者和开发者加入到Java开发和创新的行列中来，共同推动Java技术的不断发展和进步。

## 九、致谢
本报告是在我的指导老师张亮老师的悉心指导下完成的。张亮老师对我研究性课题的选题、设计、实施和撰写都给予了专业的指导和建议，使我在学习和研究中受益匪浅。在此，我向张亮老师表示最诚挚的感谢和敬意！
此外，我还要特别感谢网友AlexLiuDev233（https://github.com/AlexLiuDev233 ），他为我的研究性课题报告做出了卓越贡献。他不仅提供了宝贵的数据和资料，还给我提供了很多有用的建议和意见，帮助我完善了报告的内容和形式，并协助我开发程序。他的专业水平和热情态度让我深受鼓舞和感动。在此，我向AlexLiuDev233表示衷心的感谢和敬佩！
