### 什么是RSA
>RSA算法是一种非对称加密算法，所谓非对称，就是指该算法需要一对密钥，使用其中一个加密，则需要另一个才能解密。密钥分为公钥和私钥，私钥是自己保存，公钥提供给对方。

公钥加密 私钥解密
私钥签名 公钥验签

### RSA签名验签
>使用私钥将明文进行签名生成密文串与明文一起传输。对方收到数据后使用公钥对明文和密文串进行验签。如果验签通过就说明第一数据没有被修改过；第二这些数据一定是持有私钥的人发送的，因为私钥只有自己持有，这就起到了防抵赖的作用。

### RSA签名验签
- 工具类
```
commons-codec:commons-codec:1.8
RSAUtil.java

public class RSAUtil {
    static Logger LOG = LoggerFactory.getLogger(RSAUtil.class);

    private static final String SIGNATURE_ALGORITHM = "SHA1withRSA";                     //签名算法
    private static final String KEY_ALGORITHM = "RSA";        //加密算法RSA

    /**
     * 公钥验签
     *
     * @param text      原字符串
     * @param sign      签名结果
     * @param publicKey 公钥
     * @return 验签结果
     */
    public static boolean verify(String text, String sign, String publicKey) {
        try {
            Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
            PublicKey key = KeyFactory.getInstance(KEY_ALGORITHM).generatePublic(new X509EncodedKeySpec(Base64.decodeBase64(publicKey)));
            signature.initVerify(key);
            signature.update(text.getBytes());
            return signature.verify(Base64.decodeBase64(sign));
        } catch (Exception e) {
            LOG.error("验签失败:text={},sign={}", text, sign, e);
        }
        return false;
    }

    /**
     * 签名字符串
     *
     * @param text       需要签名的字符串
     * @param privateKey 私钥(BASE64编码)
     * @return 签名结果(BASE64编码)
     */
    public static String sign(String text, String privateKey) {
        byte[] keyBytes = Base64.decodeBase64(privateKey);
        PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(keyBytes);
        try {
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
            PrivateKey privateK = keyFactory.generatePrivate(pkcs8KeySpec);
            Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
            signature.initSign(privateK);
            signature.update(text.getBytes());
            byte[] result = signature.sign();
            return Base64.encodeBase64String(result);
        } catch (Exception e) {
            LOG.error("签名失败,text={}", text, e);
        }
        return null;
    }

}
```
- 生成密钥对
[在线生成非对称加密公钥私钥对](http://web.chacuo.net/netrsakeypair)
1024位 PKCS#8 无证书密码
- 使用演示
测试签名验签
```
/**
 * 测试加签验签
 */
public class RSAUtilTest {

    static final String publicKey = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCiqvcJQROuAmoiNQEkY0rlhaBQ\n" +
            "JqcZqdbkvhsHp5irrwvoesFKvoXELLVaS0tPyL8u5I5wfzYgr9OCjKWNZgst5Ctn\n" +
            "1V7JJp+m9xUUA+TyO2SU5Rpr5NRvYH3OZeIgBXMERGbzNZkeoLBWAq7lgQDgf9jG\n" +
            "joGFbS2VpmzuoN5yPwIDAQAB";
    static final String privateKey = "MIICdQIBADANBgkqhkiG9w0BAQEFAASCAl8wggJbAgEAAoGBAKKq9wlBE64CaiI1\n" +
            "ASRjSuWFoFAmpxmp1uS+GwenmKuvC+h6wUq+hcQstVpLS0/Ivy7kjnB/NiCv04KM\n" +
            "pY1mCy3kK2fVXskmn6b3FRQD5PI7ZJTlGmvk1G9gfc5l4iAFcwREZvM1mR6gsFYC\n" +
            "ruWBAOB/2MaOgYVtLZWmbO6g3nI/AgMBAAECgYBcteQmOhjlTCsBZARKoOzG8+ny\n" +
            "xJToY4w+wrrVGghBkXrP/Wa9GulSbcjOtasuxdNw/oLQSzCmYI/EEDUq6cXkcU8d\n" +
            "o4YtrX5kGeJZ9xDgrpB5unQSREENH5RmKhdm8ioixqO2VfSgkfUieIyUT4ilJy7A\n" +
            "/ZhT2zuIaaUo+dALQQJBAM4LdvpoWJ4UUqN4N9dmvSXWGaFwWLRmdmRNkpjKw5cy\n" +
            "D1c9HFDiGlyE8m4FZ+roL6Zmydyn33Zhn6nrGLjdiRMCQQDKGzuR9Ueo+b7b4A4P\n" +
            "I71zFpEfDZ0HZOqPgWWlhVemVk51QtzNuI/fWnUWV0R6+gJF0R2QEOgef2TDAeZc\n" +
            "uqOlAkBiZWU3JheTvj7Mo/9+1Shk5j6tMtqZpAjL06O7ZbFMBfL/hUZ9dcyC/FZN\n" +
            "pjU/IAyJWbLythRoEyzNV2Eh/2GTAkALemR1s6JwPE7UmfLydSsrQBrZ0qIaa2bO\n" +
            "46BsOBh0P+6Qxk1X+aViH/cKX8Zp3Y0HfgrZxbwJD18fnBoDJi5pAkAhRjey2F/T\n" +
            "aZrJrgGohnxLwXFpqcS68qAA8I6oEDo0kIXlvpoZ+HV4DQq3UIut87J17bNrDtlt\n" +
            "MUmSjtzBDo8x";
    @Test
    public void signTest(){
        String text = "{\"amount\":10,\"chanId\":\"111\",\"chanUserId\":\"123\",\"createAt\":\"2017-01-01 13:26:48\",\"memo\":\"string\",\"outerOrderId\":\"10001\",\"productId\":\"001\"}";
        String sign = RSAUtil.sign(text,privateKey);
        System.out.println(sign);
        System.out.println(RSAUtil.verify(text, sign, publicKey));
    }

}
```

### 验签时机
- 拦截器
- AOP

### 验签过程
- 调用方在请求头中传递authId授权编号、sign
- 使用AOP在执行实际方法前根据authId获取到公钥，进行验签
- 验签通过就继续执行

### 下单功能实现
OrderRepository
OrderService
OrderController
Swagger

### 下单功能添加RSA加签验签

### 对账介绍
>凡是涉及支付的场景都需要对账，对账是支付体系中最重要的一环，也是保证交易、资金安全的重要手段。在互联网金融行业或者电商行业中，对账其实就是确认在固定周期内和支付提供方（银行和第三方支付）的交易、资金的正确性，保证双方的交易、资金一致正确。

### 相关概念
- 轧账和平账
- 长款和漏单

### 对账文件
- order_id,outer_order_id,chan_id,chan_user_id,product_id,order_type,amount,create_at
- yyyy-MM-dd HH:mm:ss
- order_status=success
- | txt /opt/verification/{yyyy-MM-dd-chanId}.txt
 - native sql
- verification_order 

VerificationOrder.java
VerifyRepository.java
VerifyService.java
VerifyTest.java 

### 解析对账文件
VerifyService.java 
ChanEnum.java
VerifyTest.java 

### 对账
- 对账流程
- 定时执行
- 长款
- 漏单
- 不一致
模拟上述三种状态的数据
VerifyRepository.java
VerifyService.java
VerifyTest.java 

### 优化对账
- 生成、解析文件分批次
- 在备份库或者读库执行
- java程序、nosql对账

### 平账
- 收到异常对账结果
- 通知人工 邮件短信
- 轧差。。。

### 定时对账
- 固定时间自动执行
- @EnableScheduling @Scheduled
- cron表达式
task包
VerifyTask.java
cron 在线

### 定时对账问题
- 多实例部署
- 应用不可用
quartz框架可以解决

### JPA多数据源
- 主备、读写分离
- spring boot 自动配置过程
[Annotation based configuration](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#jpa.java-config)
[Data Access](https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#howto-data-access)
-- mysql主从复制
-- alibaba/otter

seller application.yml
configuration/DataAccessConfiguration
VerifyTest
需要把不同数据源的repository分开
- 总结
为什么需要多数据源
文档
dataSource / entityManagerFactory / transationManager
spring-boot-autoconfigure -> JpaRepositoriesAutoConfiguration -> 2

{JpaRepositoriesAutoConfiguration -> @EnableJpaRepositories
{HibernateJpaAutoConfiguration -> JpaBaseConfiguration -> 2

{{transactionManager
{{entityManagerFactory

@Primary


### JPA读写分离
- 不同数据源相同repositories
- 添加额外接口，继承
BackupOrderRepository
VerifyTest
- 修改源码
修改源码黑科技
RepositoryConfigurationDelegate
RepositoryBeanNamePrefix

- 接口继承，不同包，多数据源原理、
- 同一个Repository，拦截注册过程

### [TYK](https://tyk.io/)
>TYK是一个开源的、轻量级的、快速可伸缩的API网关，支持配额和速度限制，支持认证和数据分析，支持多用户多组织，提供全RESTful API。通常情况下，你只需要关心业务逻辑的实现，其他的都可以交给API网关，是用户与应用之间的一道屏障。

- Docker 安装TYK，需要key，需要把pump.conf、tyk.conf、tyk_analytics.conf中的host换成主机IP，mongo域名换成IP、tyk_dashboard换成IP、
- API详解
配置销售端swagger

- 访问控制
使用授权编码
授权编码管理
docker ps
docker exec -it {id} bash
~/portal/templates/scripes.html
docker commit {id} {name}
docker tag {name} docker.io/tykio/tyk-dashboard

- 节流限速
限速（rate limit）比如计算规则（10次/s）
第一次请求过来时，记录时间戳。从第二次请求开始，将请求到达时间与最早的一次请求时间进行比较，如果大于1s就把之前的记录全部清除，记录本次请求时间戳；如果小于1s，判断记录是否大于10次，如果小于10次，正常访问，否则返回限速错误。
配额（quotas）比如计算规则（1000次/day）
保存总次数，当前剩余次数，重置周期（s），下次重置时间。
API

- 其他功能
统计分析
webhooks
常用配置

- 架构及运行过程

- HTTPS部署过程
HTTP 
SSL/TLS 非对称加密算法 + 对称加密 + 散列算法
HTTPS HTTP+SSL/TLS
HTTPS 握手过程

- HTTPS证书
域名、公网IP
购买证书/免费 Let's Encrypt
修改服务器配置
TYK HTTPS

Springboot 部署 HTTPS
1. keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "D:\tomcat.keystore"

server:
  ssl:
    enable:true
    key-store:file:D:\tomcat.keystore
    key-password: lh123

### springboot 2.0
- 新增特性
- 代码重构
- 配置变更

升级
jdk/gradle/dependencies.gradle
配置文件
相关引用路径
