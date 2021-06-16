# fisco-bcos-spring-boot-starter

这是一个 fisco bcos 的java sdk starter，方便项目中使用sdk。
目前可使用JitPack的方式来引入。

## 在自己的项目配置使用sdk及编写相应的contract服务。

### Maven方式引入
- 在要使用的项目pom.xml中添加依赖仓库如下
```xml
    <repositories>
		<repository>
		    <id>jitpack.io</id>
		    <url>https://www.jitpack.io</url>
		</repository>
	</repositories>
```
- 在要使用的项目pom.xml中添加依赖如下
```xml
  <dependency>
	    <groupId>com.github.FISCO-BCOS</groupId>
	    <artifactId>fisco-bcos-spring-boot-starter</artifactId>
	    <version>dev-SNAPSHOT</version>
	</dependency>
```

### 或者通过Gradle方式引入
- 在要使用的项目build.gradle文件中添加依赖仓库如下
```Groovy
    allprojects {
		repositories {
			...
			maven { url 'https://www.jitpack.io' }
		}
	}
```
- 在要使用的项目build.gradle中添加依赖如下
```Groovy
	dependencies {
	        implementation 'com.github.FISCO-BCOS:fisco-bcos-spring-boot-starter:dev-SNAPSHOT'
	}
```

###  配置文件
- 添加properties配置，以application.properties中的配置为例

  *注意，把密钥目录(~/fisco/nodes/127.0.0.1/sdk)复制到bcos.cryptoMaterial.certPath定义的目录下*
```properties
# indicate BCOS SDK instance is needed
bcos.hasInstance=true
# where are key pairs
bcos.cryptoMaterial.certPath=conf

# The peer list to connect
bcos.network.peers[0]=127.0.0.1:20200
bcos.network.peers[1]=127.0.0.1:20201

# if user account is created, please config it here
bcos.account.keyStoreDir=account
bcos.account.accountFilePath=conf/0x22fec9d7e121960e7972402789868962238d8037.pem
bcos.account.accountFileFormat=pem
bcos.account.accountAddress=0x22fec9d7e121960e7972402789868962238d8037

bcos.threadPool.channelProcessorThreadSize=16
bcos.threadPool.receiptProcessorThreadSize=16
bcos.threadPool.maxBlockingQueueSize=102400
```
- 使用 `./sol2java.sh` 把写好的solidity合约转译成java class，请参考官档[地址](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/tutorial/sdk_application.html#id6)。下面范例中的`BCOSLogger`就是转译后的class。

- 定义自己的合约service，为方便使用可以继承自`AbstractContractService`,下面是一个范例：
```java
@Component
public class BCOSLoggerService extends AbstractContractService {

    static Logger logger = LoggerFactory.getLogger(BCOSLoggerService.class);

    /* 使用链上已经部署合约，需配置。 */
    @Value("${bcos.contract.logger.address}")
    private String contractAddr;

    public String getContractAddress() {
        return this.contractAddr;
    };

    public void deployContract() throws ContractException {
        contractAddr = BCOSLogger.deploy(client, cryptoKeyPair).getContractAddress();
    }

    /* block chain functions are as below. */
    public LogAsset queryLog(String logId) {
        try {
            BCOSLogger loggerContract = BCOSLogger.load(contractAddr, client, cryptoKeyPair);
            Tuple3<List<String>, List<String>, List<String>> queryResult = loggerContract.query(logId);
            return new LogAsset(queryResult.getValue1().get(0), queryResult.getValue2().get(0),
                    queryResult.getValue3().get(0));
        } catch (Exception e) {
            /* ignore */
        }
        return new LogAsset();
    }
}
```

- 接下来，就可以在需要使用的地方，直接该service的方法，来调用使用合约。

## 范例应用位于example目录下

该应用是保存日志摘要到链上，以供未来存证校验使用。范例提供2个接口和 /addlog /verifylog.启动方式如下：

- 首先在本地安装好fisco bcos [参照](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)
- 将keys（一般位于~/fisco/nodes/127.0.0.1/sdk）拷贝到 example/src/main/resources/conf 目录下
- 准备好spring-boot-starter, 如果本地使用，在项目目录下执行 `./mvnw clean install`
- 进入example，启动项目 ` ./mvnw spring-boot:run`
- 访问 `http://localhost:8080`, 点击swagger的链接，测试接口。

