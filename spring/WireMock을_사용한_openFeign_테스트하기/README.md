
## 1. 문제 상황 정의하기
진행 중인 프로젝트에서 다른 모듈의 API를 불러와야 한다.

매번 restTemplate을 사용했는데 곧 deprecated 된다길래... openFeign을 사용해보았다.

작성한 코드를 바탕으로 WireMock을 사용하여 openfeign 통합 테스트를 해볼 것이다.


## 2. OpenFeign을 사용한 client

현재 프로젝트에서 다른 모듈의 API 호출하기 위해 사용하는 openFeign 관련 로직이다.

```java=
@FeignClient(name = "processing", url = "${feign.test-api.url}")
public interface ProcessingFeignClient {

    @PostMapping("/api/confirm/test")
    ResultDto proceed(@RequestBody ProcessingRequestDto processingRequestDto);
}
```

## 3. WireMock을 사용한 테스트


### 3-1. 우선 의존성을 추가한다.

```xml=
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-wiremock</artifactId>
	<version>3.1.3</version>
</dependency>
```

### 3-2. WireMock을 사용하여 API를 호출할 모의 서버를 구축한다.

```java=
@TestConfiguration
static class WireMockConfig {

	@Bean(initMethod = "start", destroyMethod = "stop")	
	public WireMockServer mockService() {
		return new WireMockServer(9651);
	}
}
```

### 3-3. properties 파일에 @FeignClient > url에 사용할 테스트 url도 넣어준다.

```yaml=
# application-test.yaml
feign:
    test-api:
      url: localhost:9651
```


### 3-4. 모듈 API는 response로 json을 줄 것이기 때문에 test response도 json으로 만들어준다.

아래에서 보겠지만 .withBodyFile을 통해 해당 json을 불러오게 되는데 불러오는 경로의 default는 resources.__files.payload 이다. 

```json=
{
  "code": 200,
  "message": "OK"
}
```


### 3-5. 임의로 만든 모의 서버를 사용해 api 호출 시 어떤 응답이 와야하는지 만들어준다.

```java=
class ProcessingMock {
	public static void setup(WireMockServer wireMockServer) {
    	wireMockServer.stubFor(WireMock.post(WireMock.urlMatching("/api/confirm/test"))
				.willReturn(WireMock.aResponse()
					.withStatus(HttpStatus.OK.value())
                	.withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                	.withBodyFile("payload/get-response.json")
                )
       );
   }
}
```

### 3-6. 마지막으로 원하는 테스트를 작성하면 된다.

```java=
@Test
public void api_요청_성공_테스트() {
    assertThat(processingFeignClient.proceedRegulation(processingRequestDto)
    		.getCode()).isEqualTo(HttpStatus.OK.value());
    assertThat(processingFeignClient.proceed(processingRequestDto)
            .getMessage()).isEqualTo("OK");
}
```


## 4. 전체 테스트 코드

```java=
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {WireMockConfig.class})
@EnableConfigurationProperties
@ActiveProfiles("test")
@SpringBootTest
class processingFeignClientTest {

    @Autowired
    private WireMockServer wireMockServer;

    @Autowired
    private ProcessingFeignClient processingFeignClient;

    private ProcessingRequestDto processingRequestDto;

    private ProcessingMock processingMock;

    @BeforeEach
    public void setup() {
        processingMock.setup(wireMockServer);
        processingRequestDto = ProcessingRequestDto.builder()
                .key("test-contentKey")
                .tag(Tag.SUCCESS)
                .collection(Collection.VIEW)
                .code(1023)
                .build();
    }

    @Test
    public void api_요청_성공_테스트() {
        assertThat(processingFeignClient.proceed(processingRequestDto)
                .getCode()).isEqualTo(HttpStatus.OK.value());
        assertThat(processingFeignClient.proceed(processingRequestDto)
                .getMessage()).isEqualTo("OK");
    }

    @TestConfiguration
    static class WireMockConfig {

        @Bean(initMethod = "start", destroyMethod = "stop")
        public WireMockServer mockService() {
            return new WireMockServer(9651);
        }
    }

    static class ProcessingMock {
        public static void setup(WireMockServer wireMockServer) {
            wireMockServer.stubFor(WireMock.post(WireMock.urlMatching("/api/confirm/test"))
                    .willReturn(WireMock.aResponse()
                            .withStatus(HttpStatus.OK.value())
                            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
                            .withBodyFile("payload/get-response.json")
                    )
            );
        }
    }

}
```



