---
description: 效能調校
---

# 微服務系統效能調校

## 環境配置：

APM 、關閉日誌（LOG)&#x20;

application.properties 中設定：

```
logging.level.com.esb.wcp=INFO/OFF
```

## 調校步驟：

1. 確認打測基準量、固定環境參數、單一打測腳本
2. 進行打測
3. 觀察APM是否出現長時間(>1000mm)運行步驟、間距過大的偏移、服務的記憶體、執行緒、CPU用量
4. 分析該運行步驟是否合理、比對程式碼是否存在可以重構為同步執行之步驟
5. 調整程式、參數
6. 重新打測，重複步驟３～６ ，直到達成目標

## APM異常範例：

![](<.gitbook/assets/image (2).png>)：步驟間有長空白

![](<.gitbook/assets/image (3).png>)：步驟間有偏移

![](<.gitbook/assets/image (4).png>)：步驟間有排隊

## 微服務系統效能調校要點：

### Restful API 連線池化大小

#### 判斷依據為APM中顯示出應同步發送之服務請求，出現排隊、發送前空白過長或偏移現象。

* 因為微服務系統多半使用各式Restful API進行資料處理，因此需要通用的連線池，避免每次對同一DNS連線都需要建立全新的連線，耗費成本。
*   application.properties 池化設定參數如下：



    ```
    maxTotal 
    最大連接總數

    defaultMaxPerRoute 
    對每個目標Domain最大連接數，例如：若設定為２，連線到Risk和Customer可以建立的連線就只有各兩條，Risk2條、Customer2條，超過的需要排隊等待這兩條。

    connectTimeout
    創建連接的逾期時間

    connectionRequestTimeout
    取得連線池連線的逾期時間

    socketTimeout
    資料逾期時間

    staleConnectionCheckEnabled
    檢核連線是否可用
    ```

### 資料庫連接池化大小

#### 判斷依據為APM中顯示應連線至資料庫的步驟前方有空白過長、偏移；Log中出現資料庫連線失敗。

* 因為微服務若使用同一資料庫，不像過往單一服務共用連線池的概念，會變成每個服務都有自己的連線池，故所需的連線數會大於單一服務，所以需要注意連線池的量。
* 若有遇到連線資料庫session數不足或連線失敗，是因為連線池的連線並無法正常釋放連線，而連線池設計成目的就是為提供連線能共用化，減少建立連線與關閉連線所需的成本，但這個訊息出現，應首先識別是否有連線無法正常釋放的現象，常見的有下列情境。
  * 未使用Spring JdbcTemplate 共用連線池，沒有適時關閉連線。
  * 讀寫不分離，導致讀取所用的連線需要等待寫入連線完成，整體交易事務生命週期走完才能正常釋放。
  * 交易事務控管過程夾帶外部資訊源的收集，導致資料庫操作過程被外部資訊源回應時間綁定無法即時釋放連線。
  * 服務引入spring-boot-starter-data-jpa套件，導致預設開啟 spring.jpa.open-in-view，使連線將無法正常釋放，請於服務application.properties中設定 spring.jpa.open-in-view=false 即可。
* application.properties 池化設定參數如下：

```
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.hikari.schema=WCP

spring.datasource.hikari.connection-test-query=select 1 from dual
連線前透過該設定驗證連線是否正常，

spring.datasource.hikari.connection-timeout=30000
等待獲得連線至逾期所需時間

spring.datasource.hikari.minimum-idle=5
最小待命連線數

spring.datasource.hikari.maximum-poolSize=30
最大連線數

spring.datasource.hikari.idle-timeout=600000
待命連線逾期所需時間

spring.datasource.hikari.max-lifetime=1800000
連線最長逾期時間

spring.jpa.database=oracle
spring.jpa.open-in-view=false
```

有關於最小待命連線數及最大待命連線數，在效能調校時需注意，兩者間的差會導致連線數需要不斷上下游移，故建議不設定最小待命連線數，使其連線數保持在最大的連線數，節省服務過程中需要連線數的建立與關閉成本。

關於Oracle DB連線數的設定值參考，建議是設定在８～３０之間，若大於此數請確認是否有上述的情境，導致連線數不足，因為官方有說明連線數越小處理速度越快，並非開大連線數才可以加快執行速度。

### 多執行緒（平行處理）池化大小

#### 判斷依據為APM上顯示該步驟後有其他非依賴之步驟正在排隊、調整成同步進行卻發現有偏移、有長段空白。

* 若有需要進行平行處理的步驟，如：同步發查多個外部系統取回資訊，則需要建立執行緒池，透過該執行緒池進行執行緒的管理。
* 若發現平行處理的步驟無正常平行處理，需檢查下列項目：
  * 快取annotation與非同步annotation落於相同class中，需切分至不同層級。
  * 非同步annotation方法中有呼叫其他方法，但該方法也存在同一個非同步annotation，需要額外建立一個多執行緒池，分割成不同的非同步annotation，不能共用同一組。
* 若有使用spring cache，可以將Annotation的讀取紀錄輸出至Log，可以確認其執行緒建置方式。

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.springframework.cache" level="trace">
        <appender-ref ref="STDOUT" />
    </logger>
</configuration>
```

* application.properties 池化設定參數如下：

```
server.async.corePoolSize=1000
執行緒池基本大小
server.async.maxPoolSize=1000
執行緒池中最大執行緒數
server.async.queueCapacitySize=20000
執行緒池可允許之緩衝區大小
```



### 長短快取設計、初始化快取、H2遠程、H1記憶體快取配置

#### 判斷依據為APM需要長時間花費在初始資料建置、調用異動頻率低卻耗時的跨模組資料存取。

* 建立lettuce Redis 連線工廠，共用連線至Redis伺服器。
* 建立spring boot cache 並使用上述連線工廠 ，設計Annotation管理快取的生命週期、命名規則。
* 建立RedisTemplate ，使初始化作業能夠手動進行快取建置。
* 透過spring Annotation設計短快取，使服務間可以透過Restful API 的trace id 使用H2快取資料內容。
* 透過@Autowired 注入RedisTemplate 。
* 若發現交易場景需要多個Restful API來完成一個交易，需思考是否將所需的資料提前至準備至快取中，減少後續Restful API 需要耗費成本取得相同資料。
* 需思考是否需要建立快取生命週期延展功能(到期自動重取)。
* application.properties 設定參數如下：

```
### -------Redis config----------
# Redis db index
spring.redis.database=0
# Redis server ip
spring.redis.host=OOXX.testeXXOO.com.tw
# Redis server port
spring.redis.port=6379
# Redis server password
spring.redis.password=
# Redis connect pool max (native = no limit)
spring.redis.pool.max-active=-1
# Redis connect pool wait time (native = no limit)
spring.redis.pool.max-wait=-1
# Redis connect pool idle max
spring.redis.pool.max-idle=300
# Redis connect pool idle min
spring.redis.pool.min-idle=200
# Redis connect timeout
spring.redis.timeout=300000
spring.redis.cluster.nodes=OOXX4U.testeXXOO.com.tw:6379,OOXX5U.testeXXOO.com.tw:6379,XXOO6U.testeOOXX.com.tw:6379
spring.redis.cluster.max-redirects=12
```

* 透過@Configuration 觸發服務啟動時建立H2遠程快取，降低跨模組取資料的成本，如下列範例。

```java
@Configuration
public class ProdInfoCache {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private ProductCtgService productCtgService;

    @Bean
    void generatorProdInfoCache() throws Exception {
        String fundPrdLst = StringUtil.object2Json(productCtgService.getFundPrdLst(new RequestDTO()).getBodyDTO(), ArrayList.class);
        String bondPrdLst = StringUtil.object2Json(productCtgService.getBondPrdLst(new RequestDTO()).getBodyDTO(), ArrayList.class);
        String stockPrdLst = StringUtil.object2Json(productCtgService.getEtfStkPrdLst(new RequestDTO()).getBodyDTO(), ArrayList.class);

        String stkLvlRange = StringUtil.object2Json(productCtgService.getStockLevelRange(new RequestDTO()).getBodyDTO(), StkLvlRangeInfoDTO.class);
        ObjectMapper rsrMapper = new ObjectMapper().configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // Feature that determines whether it is acceptable to coerce single value array (in JSON) values to the corresponding value type.
        rsrMapper.enable(DeserializationFeature.UNWRAP_SINGLE_VALUE_ARRAYS);
        StkLvlRangeInfoDTO stockLevelRange = rsrMapper.readValue(stkLvlRange, new TypeReference<StkLvlRangeInfoDTO>() {
        });
        ArrayList<FundPrdInfoDTO> fundPrdInfoDTOS = rsrMapper.readValue(fundPrdLst, new TypeReference<ArrayList<FundPrdInfoDTO>>() {
        });
        ArrayList<BondPrdInfoDTO> bondPrdInfoDTOS = rsrMapper.readValue(bondPrdLst, new TypeReference<ArrayList<BondPrdInfoDTO>>() {
        });
        ArrayList<StockPrdInfoDTO> stockPrdInfoDTOS = rsrMapper.readValue(stockPrdLst, new TypeReference<ArrayList<StockPrdInfoDTO>>() {
        });

        redisTemplate.opsForValue().set("RiskService:FundPrdLstInfo.FundPrdLstInfo", fundPrdInfoDTOS, 8, TimeUnit.HOURS);
        redisTemplate.opsForValue().set("RiskService:BondPrdLstInfo.BondPrdLstInfo", bondPrdInfoDTOS, 8, TimeUnit.HOURS);
        redisTemplate.opsForValue().set("RiskService:StockPrdLstInfo.StockPrdLstInfo", stockPrdInfoDTOS, 8, TimeUnit.HOURS);
        redisTemplate.opsForValue().set("RiskService:StkLvlRangeInfo.StkLvlRangeInfo", stockLevelRange, 8, TimeUnit.HOURS);

    }
}
```

* 透過@Configuration 觸發服務啟動時建立sigleton H1快取並注入到對應參數中，降低每次初始化需要花費的成本，如下列範例。

<pre class="language-java"><code class="lang-java"><strong>@SpringBootApplication
</strong>@ImportResource(locations = {"classpath:/config/applicationContext.xml"})
@ComponentScan("com.esb")
@EnableCaching
@EnableTransactionManagement
public class RiskServiceApplication extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
      return application.sources(RiskServiceApplication.class);
   }

   public static void main(String[] args) throws Exception {
      SpringApplication app = new SpringApplication(RiskServiceApplication.class);
      ConfigurableApplicationContext ctx = app.run(args);
      //針對啟動當下先行建立規則引擎檢核模板
      Map&#x3C;String, KieBaseTemplateVO> kieBaseTemplateMap = (Map&#x3C;String, KieBaseTemplateVO>) ctx.getBean("KieBaseTemplateMap");
      TxServiceImpl txService = (TxServiceImpl) ctx.getBean("TxServiceInterface");
      txService.setKieBaseTemplateMap(kieBaseTemplateMap);
   }

}
</code></pre>

### 限流設計

#### 判斷依據為APM中發現有異常伺服器重啟、服務取用異常、服務被OCP重啟。

* 確認是否server.undertow.work-threads 設定過小
* 透過AOP建立RateLimiter(令牌桶限流演算法) ，限制Controller 接收的請求量，預留通道給平台進行服務health check。
* application.properties 設定參數如下：

<pre><code>server.undertow.io-threads=1
<strong>I/O讀寫執行緒數
</strong>server.undertow.worker-threads=1024
可執行之請求執行緒數
</code></pre>

* 範例代碼如下：

```java
@Component
@Aspect
@Profile({"dev", "ocps", "ocpu", "sit", "uat", "ocpp", "prod"})
@Order(INBOUND_TRACE_LOG_ASPECT_ORDER)
public class ControllerRateLimiterAspect {
    /**
     * 限流令牌總數
     */
    @Value("${server.undertow.worker-threads}")
    private String threads;
    RateLimiter rl;

    @Bean
    void setRateLimiter() {
        this.rl = RateLimiter.create(Integer.parseInt(this.threads) - 24);
    }

    /**
     * Controller切入點
     */
    @Pointcut("execution(com.sss.W.core.message.RestServiceResponse com.sss.w.*.controller..*(..))")
    public void controllerCut() {
    }

    /**
     * Risk Controller切入點
     */
    @Pointcut("execution(com.sss.w.core.message.RestServiceResponse com.sss.w.*.transcation.controller..*(..))")
    public void riskControllerCut() {

    }

    /**
     * 撥入電文前進確認限流數
     *
     * @param joinPoint 擷取點
     * @param <T>       泛型物件
     * @return RestServiceResponse
     * @throws Throwable 錯誤訊息
     */
    @Around("(controllerCut()) || (riskControllerCut())")
    public <T> RestServiceResponse aroundContoller(ProceedingJoinPoint joinPoint) throws Throwable {

        /**
         * 限流令牌，等同work thread數-預留數（24）。
         * ##預留數是給平台監測使用。
         */

        RestServiceResponse response = null;
        //嘗試取得令牌，若沒取得，將等待兩秒再取一次，若還是無法取得將會拋出例外。
        if (rl.tryAcquire()) {
            response = (RestServiceResponse) joinPoint.proceed();
        } else {
            throw new WCPException("SSS-S006", "後端伺服器忙碌中");
        }
        return response;
    }

}
```

### 執行過久的預存程序

#### 判斷依據為APM上呼叫該執行緒執行時間過長(>1000mm)、順間請求量大增時出現長空白。

* 確認該預存程序是否有lock議題。
* 確認該預存程序是否有index議題。

### 讀寫服務是否可分離、Transcational  交易事務控管是否合乎成本

#### 判斷依據為APM上有步驟間有長空白等待、資料庫Session不足訊息。

* 分析該交易流程是否必要使用@Transcational，若為讀取則應該移除該Annotation。
* 分析該交易流程是否可以調整將讀取部分隔離不受交易事務控管。
* 分析該交易流程是否涉及外部資訊和資料寫入，若有請隔離外部資訊讀取，使讀取不受交易事務控管，降低交易事務需要等待外部資訊回應時間成本，佔用資料寫入連線資源。

