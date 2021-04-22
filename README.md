Springboot-Webproject-ChatService-개망톡
=====================

참여인원 : 백앤드 2명 (김남석, 채수연) / 프론트 2명 (전재민, 조윤경)  


</br>

프로젝트 API 서버 주소:  
프로젝트 웹 서버 주소: http://gaemangtalk.site/  
프론트엔드 레포지토리 주소: https://github.com/greedysiru/gaemangtalk_frontend


</br>

개발 언어
---------
- Backend: Java 8
- Frontend: React


</br>

개발 환경
---------
- Java: JDK 1.8.0  
- IDE: IntelliJ IDEA 2020.3.3 x64  
- Build Management: Gradle  
- Framework: SpringBoot  
> - ORM: Spring Data JPA  
> - Sub-Framework: Spring security  
> - Other Libraries: Websocket, AWS, Lombok

- DBMS
> - MySql : 데이터베이스
> - Redis : 메시징큐 기능 사용
> - S3 : 이미지 스토리지

</br>

폴더 구조
---------
```
main
java
com.clone.daangnclone
│  │  
├─ config
│  └─ EmbeddedRedisConfig.java
│  └─ JwtAuthenticationFilter.java
│  └─ JwtTokenProvider.java
│  └─ RedisConfig.java
│  └─ WebMVCConfig.java
│  └─ WebSecurityConfig.java
│  └─ WebSocketConfig.java
│  │  
├─ controller
│  └─ ChatMessageController.java
│  └─ ChatRoomController.java
│  └─ S3Controller.java
│  └─ UserController.java
│  │  
├─ dto
│  └─ ChatMessageRequestDto.java
│  └─ ChatRoomRequestDto.java
│  └─ UserLoginRequestDto.java
│  └─ UserProfileRequestDto.java
│  └─ UserSignupRequestDto.java
│  │  
├─ error
│  └─ ErrorCode.java
│  └─ ErrorResponse.java
│  └─ ExceptionController.java
│  │  
├─ handler
│  └─ CustomMessageResponse.java
│  └─ StompHandler.java
│  │  
├─ kakao
│  └─ KakaoOAuth2.java
│  └─ KakaoUserInfo.java
│  │ 
├─ models
│  └─ ChatMessage.java
│  └─ ChatRoom.java
│  └─ Timestamped.java
│  └─ User.java
│  └─ UserDetailsImpl.java
│  └─ UserRole.java
│  │ 
├─ pubsub
│  └─ RedisSubscriber.java
│  │ 
├─ repository
│  └─ ChatMessageRepository.java
│  └─ ChatRoomRepository.java
│  └─ UserRepository.java
│  │ 
├─ service
│  └─ ChatRoomService.java
│  └─ ChatMessageService.java
│  └─ UserDetailsServiceImpl.java
│  └─ UserService.java
│  │ 
├─ utils
│  └─ CORSFilter.java
│  └─ MailUtil.java
│  └─ S3Uploader.java
│  └─ Uploader.java
│  │ 
└─  ChatserviceApplication.java
```


</br>

주요 기능
--------
- 회원가입 페이지
> - 이메일 중복 확인
> - 닉네임 유효성 검사
> - 비밀번호 일치 확인


- 비밀번호 찾기 페이지
> - 이메일 입력을 통한 인증번호 발송
> - 인증번호 입력
> - 인증 성공 시 비밀번호 변경 페이지 이동


- 마이프로필 페이지
> - 유저 프로필 사진 : 사진 파일을 업로드 하면 S3 이미지 스토리지에 저장한 후 URL 반환
> - 유저 정보 수정 : 유저 프로필 사진, 유저 닉네임 수정


- 채팅 페이지
> - 채팅방 생성하기 : 채팅방 이름, 대표 이미지, 카테고리 설정 가능
> - 채팅방 조회하기 : 이름, 이미지, 카테고리에 추가하여 생성자 프로필 정보 조회
> - 채팅방 카테고리별 조회하기 : 채팅방 카테고리에 따라 해당 카테고리를 가진 채팅방 목록 조회
> - 채팅방 입장
> > - 해당 채팅방에서 기존에 대화했던 대화 내용 전체 조회
> > - 채팅방 목록 중 한 곳을 누르면 username + 님이 방에 입장했습니다 메시지 반환
> > - 해당 채팅방 Topic이 구독되어, 해당 채팅방으로 pub 되는 메세지를 모두 웹소켓을 통해 받아 볼 수 있음
> - 채팅 메시지 작성 : 채팅 메시지를 작성해 pub 하면 해당 채팅방을 구독 중인 이용자 모두에게 sub 메시지 반환
> - 채팅방 퇴장 : 다른 채팅방으로 옮겨갈 시 username + 님이 퇴장했습니다 메시지 반환


</br>

Entity Table Structure
----------------------
(업로드 예정)


</br>

API Structure
-------------
(업로드 예정)


</br>

# 기능 상세 소개


</br>

채팅방 관련 기능
-----------------
### ChatMessage
```java
@Getter
@Entity
@NoArgsConstructor
public class ChatRoom extends Timestamped {

    @Id
    @GeneratedValue( strategy = GenerationType.IDENTITY)
    @Column
    private Long id;

    @Column
    private String chatRoomName;

    @Column
    private String chatRoomImg;

    @ElementCollection
    private Set<String> category;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "user_id")
    private User user;

    public ChatRoom(ChatRoomRequestDto requestDto, UserService userService) {
        this.chatRoomName = requestDto.getChatRoomName();
        this.chatRoomImg = requestDto.getChatRoomImg();
        this.category = requestDto.getCategory();
        this.user = userService.findById(requestDto.getUserId());
    }

}
```
- 채팅방 이름, 채팅방 대표 이미지, 채팅방 카테고리 항목을 칼럼으로 만들었습니다. 
- 채팅방을 생성한 사용자 정보를 가져오기 위해 @ManyToOne으로 User Entity와 연관관계를 설정하였습니다.
- ```@ElementCollection``` : 카테고리를 여러 개 리스트로 가져오기 위한 설정
- ```@ManyToOne```: User 객체 하나에 여러 ChatRoom 객체가 연관될 수 있다는 연관관계 설정

### ChatRoomRepository
```java
public interface ChatRoomRepository extends JpaRepository<ChatRoom, Long> {
    List<ChatRoom> findAllByOrderByCreatedAtDesc();
    List<ChatRoom> findByCategory(String category);
}
```
- ChatRoom 정보는 MySql DB에 저장되며, DB와 통신을 용이하기 위해 JpaRepository를 상속하였습니다.
- findById와 같은 기본적인 명령어 외에 사용하고 싶은 명령어들을 입력해두었습니다.
- 채팅방을 모두 조회해 생성일자로 정렬, 카테고리별로 검색 두 가지 명령어를 추가하였습니다.

### CharRoomService
```java
    @Service
    public class ChatRoomService {
    
        @Resource(name = "redisTemplate")
        private HashOperations<String, String, String> hashOpsEnterInfo;
    
        private final ChatRoomRepository chatRoomRepository;
        private final UserService userService;
        public static final String ENTER_INFO = "ENTER_INFO"; // 채팅룸에 입장한 클라이언트의 sessionId 와 채팅룸 id 를 맵핑한 정보 저장
    
        @Autowired
        public ChatRoomService(ChatRoomRepository chatRoomRepository, UserService userService) {
            this.chatRoomRepository = chatRoomRepository;
            this.userService = userService;
        }
    
        // 채팅방 생성
        public ChatRoom createChatRoom(ChatRoomRequestDto requestDto) {
            ChatRoom chatRoom = new ChatRoom(requestDto, userService);
            chatRoomRepository.save(chatRoom);
            return chatRoom;
        }
    
        // 전체 채팅방 조회
        public List<ChatRoom> getAllChatRooms() {
            return chatRoomRepository.findAllByOrderByCreatedAtDesc();
        }
    
        // 카테고리별 채팅방 조회
        public List<ChatRoom> getAllChatRoomsByCategory(String category) {
            return chatRoomRepository.findByCategory(category);
        }
    
        // 개별 채팅방 조회
        public ChatRoom getEachChatRoom(Long id) {
            ChatRoom chatRoom = chatRoomRepository.findById(id).orElseThrow(
                    () -> new IllegalArgumentException("찾는 채팅방이 존재하지 않습니다.")
            );
            return chatRoom;
        }
    
        // 유저가 입장한 채팅방 ID 와 유저 세션 ID 맵핑 정보 저장
        public void setUserEnterInfo(String sessionId, String roomId) {
            hashOpsEnterInfo.put(ENTER_INFO, sessionId, roomId);
        }
    
        // 유저 세션으로 입장해 있는 채팅방 ID 조회
        public String getUserEnterRoomId(String sessionId) {
            return hashOpsEnterInfo.get(ENTER_INFO, sessionId);
        }
    
        // 유저 세션정보와 맵핑된 채팅방 ID 삭제
        public void removeUserEnterInfo(String sessionId) {
            hashOpsEnterInfo.delete(ENTER_INFO, sessionId);
        }
    
    }
```
- 채팅방 관련 비즈니스 로직들을 한 눈에 볼 수 있게 모아두었습니다.
- Controller에 바로 Repository를 의존성 주입하여 Service Layer를 안거치고 사용 가능한 로직들도 있으나,
Service Layer를 거치도록 하여 ChatRoom 관련 비즈니스 로직들을 Service Layer에서 한 눈에 볼 수 있도록 하였습니다.
  
### ChatRoomController
```java
    @CrossOrigin (origins = "*")
    @RestController
    @RequestMapping("/api/chat")
    public class ChatRoomController {
    
        private final ChatMessageService chatMessageService;
        private final ChatRoomService chatRoomService;
        private final Uploader uploader;
    
        @Autowired
        public ChatRoomController(ChatMessageService chatMessageService, ChatRoomService chatRoomService, Uploader uploader) {
            this.chatMessageService = chatMessageService;
            this.chatRoomService = chatRoomService;
            this.uploader = uploader;
        }
    
        // 채팅방 생성
        @PostMapping("/rooms")
        public ChatRoom createChatRoom(@RequestBody ChatRoomRequestDto requestDto, @AuthenticationPrincipal UserDetailsImpl userDetails) throws IOException {
            requestDto.setUserId(userDetails.getUser().getId());
            ChatRoom chatRoom = chatRoomService.createChatRoom(requestDto);
            return chatRoom;
        }
    
        // 전체 채팅방 목록 조회
        @GetMapping("/rooms")
        public List<ChatRoom> getAllChatRooms() {
            return chatRoomService.getAllChatRooms();
        }
    
        // 채팅팅방 카테고리별 조회
        @GetMapping("/rooms/search/{category}")
        public List<ChatRoom> getChatRoomsByCategory(@PathVariable String category) {
            return chatRoomService.getAllChatRoomsByCategory(category);
        }
    
        // 채팅방 상세 조회
        @GetMapping("/rooms/{roomId}")
        public ChatRoom getEachChatRoom(@PathVariable Long roomId) {
            return chatRoomService.getEachChatRoom(roomId);
        }
    
        // 채팅방 내 메시지 전체 조회
        @GetMapping("/rooms/{roomId}/messages")
        public Page<ChatMessage> getEachChatRoomMessages(@PathVariable String roomId, @PageableDefault Pageable pageable) {
            return chatMessageService.getChatMessageByRoomId(roomId, pageable);
        }
    
    }
```
- 하나의 POST 요청, 네 개의 GET 요청을 관리하는 Controller 입니다.
- 메소드 명만 보고도 최대한 해당 메소드가 처리하는 로직을 파악할 수 있게 작명하였습니다.
- ```@PostMapping``` : POST 요청
- ```@GetMapping``` : GET 요청


</br>

채팅 메시지 관련 기능
------------------
1. Spring 에서 제공해주는 웹소켓 모듈을 활용하였습니다.
2. Redis는 RabbitMQ나 Kafka 같이 전문적인 메시징 시스템의 pub/sub처럼 고도화된 기능을 제공하지는 않지만, 
   Memory DB 특성을 살려 단순하지만 가볍고 빠른 pub/sub 기능을 제공해줍니다. 
   해당 프로젝트에서는 Spring-Data-Redis 모듈을 로드하여 pub/sub를 구현하였습니다.


</br>

### WebSocketConfig
```java
    @Configuration
    @EnableWebSocketMessageBroker
    public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
        private final StompHandler stompHandler;
    
        @Autowired
        public WebSocketConfig (StompHandler stompHandler) {
            this.stompHandler = stompHandler;
        }
    
        @Override
        public void configureMessageBroker(MessageBrokerRegistry config) {
            config.enableSimpleBroker("/sub"); // prefix /sub 로 수신 메시지 구분
            config.setApplicationDestinationPrefixes("/pub"); // prefix /pub 로 발행 요청
        }
    
        @Override
        public void registerStompEndpoints(StompEndpointRegistry registry) {
            registry.addEndpoint("/chatting") // url/chatting 웹 소켓 연결 주소
                    .setAllowedOriginPatterns("*://*")
                    .withSockJS(); // sock.js를 통하여 낮은 버전의 브라우저에서도 websocket 이 동작할수 있게 한다
        }
    
        // StompHandler 인터셉터 설정
        // StompHandler 가 Websocket 앞단에서 token 및 메시지 TYPE 등을 체크할 수 있도록 다음과 같이 인터셉터로 설정한다
        @Override
        public void configureClientInboundChannel(ChannelRegistration registration) {
            registration.interceptors(stompHandler);
        }
    
    }
```
- ```WebSocketMessageBrokerConfigurer```로 ```WebSocketConfig``` 클래스를 구현하였습니다. 웹 소켓 연결을 구성하기 위한 메서드들을 구현하고 제공합니다.
- ```registerStompEndpoints```에서 웹소켓 통신을 담당할 endpoint를 지정합니다. ```setAllowedOriginPatterns("*://*")```을 통해 CORS 이슈를 해결합니다.
  ```withSockJS```는 웹 소켓을 지원하지 않는 브라우저에 폴백 옵션을 활성화하는 데 사용됩니다.
- ```configureMessageBroker``` 메서드는 한 클라이언트에서 다른 클라이언트로 메시지를 라우팅 하는데 하용될 메시지 브로커를 구성합니다.
- ```setApplicationDestinationPrefixes```는 client의 ```SEND```요청을 처리합니다.
- ```enableSimpleBroker```는 해당 경로로 ```SimpleBroker```를 등록합니다.
```SimpleBroker```는 해당하는 경로를 ```SUBSCRIBE```하는 client에게 메시지를 전달하는 작업을 수행합니다.
- ```@Configuration``` : 클래스 선언 앞에 작성. 해당 클래스가 Bean 설정을 할 것이라는 것을 암시. ```@Component```를 포함하는 어노테이션
- ```@EnableWebSocketMessageBroker``` : WebSocket 서버를 활성화 하는데 사용
- 해당 웹소켓 통신은 STOMP 프로토콜을 사용합니다.

### RedisConfig
```java
@Configuration
public class RedisConfig {

    // 어플리케이션에서 사용할 redisTemplate 설정
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));
        return redisTemplate;
    }

    // 단일 Topic 사용을 위한 Bean 설정
    @Bean
    public ChannelTopic channelTopic() {
        return new ChannelTopic("chatroom");
    }

    // redis에 발행(publish)된 메시지 처리를 위한 리스너 설정
    @Bean
    public RedisMessageListenerContainer redisMessageListener(RedisConnectionFactory connectionFactory,
                                                              MessageListenerAdapter listenerAdapter,
                                                              ChannelTopic channelTopic) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.addMessageListener(listenerAdapter, channelTopic);
        return container;
    }

    // 실제 메시지를 처리하는 subscriber 설정 추가
    @Bean
    public MessageListenerAdapter listenerAdapter(RedisSubscriber subscriber) {
        return new MessageListenerAdapter(subscriber, "sendMessage");
    }

}
```
- Bean을 통해 어플리케이션에서 사용할 RedisTemplate을 설정합니다.
- ```redisMessageListner```로 redis에 /pub로 발행된 메시지가 들어오면 이에 반응하여 메시지를 처리합니다.
- ```listenerAdapter```는 발행된 메시지가 들어오면 해당 메시지를 구독자들에게 보내는 과정을 설정합니다. ```sendMessage``` 메서드를 통해 구독자들에게 메시지를 보내게 되는데, 해당 메서드는 ```RedisSubscriber```에 구현되어 있습니다.

### RedisSubscriber
```java
// Redis 에서 메시지가 발행(publish)되면 대기하고 있던 Redis Subscriber 가 해당 메시지를 받아 처리한다.
    public void sendMessage(String publishMessage) {
        try {
            // ChatMessage 객채로 맵핑
            ChatMessage chatMessage = objectMapper.readValue(publishMessage, ChatMessage.class);
            // 채팅방을 구독한 클라이언트에게 메시지 발송
            messagingTemplate.convertAndSend("/sub/api/chat/rooms/" + chatMessage.getRoomId(), chatMessage);
        } catch (Exception e) {
            log.error("Exception {}", e);
        }
    }
```
- ```sendMessage``` 메서드를 구현한 ```RedisSubscriber``` 클래스입니다. 
- /sub/api/chat/rooms/{roomId} 토픽을 ```SUBSCRIBE``` 중인 모든 사용자들에게 보내도록 설정해두었습니다.
- 이제 기본 설정들을 마쳤으니 클라이언트에서 오는 메시지의 흐름 순서대로 코드를 보여드리겠습니다.

### 1. StompHandler
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class StompHandler implements ChannelInterceptor {

    private final JwtTokenProvider jwtTokenProvider;
    private final ChatRoomService chatRoomService;
    private final ChatMessageService chatService;

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);

        if (StompCommand.CONNECT == accessor.getCommand()) { // websocket 연결요청
            String jwtToken = accessor.getFirstNativeHeader("token");
            log.info("CONNECT {}", jwtToken);
            // Header의 jwt token 검증
            jwtTokenProvider.validateToken(jwtToken);
        }

        else if (StompCommand.SUBSCRIBE == accessor.getCommand()) { // 채팅룸 구독요청
            // header정보에서 구독 destination정보를 얻고, roomId를 추출한다.
            // roomId를 URL로 전송해주고 있어 추출 필요
            String roomId = chatService.getRoomId(Optional.ofNullable((String) message.getHeaders().get("simpDestination")).orElse("InvalidRoomId"));

            // 채팅방에 들어온 클라이언트 sessionId를 roomId와 맵핑해 놓는다.(나중에 특정 세션이 어떤 채팅방에 들어가 있는지 알기 위함)
            // sessionId는 정상적으로 들어가고있음
            String sessionId = (String) message.getHeaders().get("simpSessionId");
            chatRoomService.setUserEnterInfo(sessionId, roomId);

            // 클라이언트 입장 메시지를 채팅방에 발송한다.(redis publish)
            String token = Optional.ofNullable(accessor.getFirstNativeHeader("token")).orElse("UnknownUser");
            String name = jwtTokenProvider.getAuthenticationUsername(token);
            chatService.sendChatMessage(ChatMessage.builder().type(ChatMessage.MessageType.ENTER).roomId(roomId).sender(name).build());

            log.info("SUBSCRIBED {}, {}", name, roomId);
        }

        else if (StompCommand.DISCONNECT == accessor.getCommand()) { // Websocket 연결 종료

            // 연결이 종료된 클라이언트 sesssionId로 채팅방 id를 얻는다.
            String sessionId = (String) message.getHeaders().get("simpSessionId");
            String roomId = chatRoomService.getUserEnterRoomId(sessionId);

            // 클라이언트 퇴장 메시지를 채팅방에 발송한다.(redis publish)
            String token = Optional.ofNullable(accessor.getFirstNativeHeader("token")).orElse("UnknownUser");

            if(accessor.getFirstNativeHeader("token") != null) {
                String name = jwtTokenProvider.getAuthenticationUsername(token);
                chatService.sendChatMessage(ChatMessage.builder().type(ChatMessage.MessageType.QUIT).roomId(roomId).sender(name).build());
            }

            // 퇴장한 클라이언트의 roomId 맵핑 정보를 삭제한다.
            chatRoomService.removeUserEnterInfo(sessionId);
            log.info("DISCONNECT {}, {}", sessionId, roomId);
        }
        return message;
    }

}
```
- 클라이언트에서 ```CONNECT```, ```SUBSCRIBE```, ```SEND```, ```DISCONNECT``` 요청이 오게 되면 가장 먼저 
```ChannelInterceptor```를 구현한 ```StompHandler```에 도착하게 됩니다.
- 이 곳에서 Command 요청이 어떤 것인지를 검사하고, ```CONNECT``` 요청에 한해서는 jwt token 유효성 검사를 진행하고,
```SUBSCRIBE```, ```DISCONNECT``` 요청에 한해서는 메시지 TYPE을 ENTER, QUIT으로 바꾸어 입퇴장 메시지를 발송하는 역할을 합니다.
- 특히 ```DISCONNECT```의 경우 클아이언트에서 요청을 하지 않아도 창이 닫히거나, 로그아웃 등의 액션이 일어나면 ```ChannelInterceptor```
에서 자동으로 ```DISCONNECT``` 시키는데 이때는 클라이언트에서 token 값을 보내주지 않습니다. 따라서 퇴장 메시지에 username이 뜨지 않게되어
Unknown User로 퇴장 메시지가 발송되는 것을 막기 위해 분기처리를 해놓았습니다.
  

### 2. ChatMessageController
```java
    // 채팅 메시지를 @MessageMapping 형태로 받는다
    // 웹소켓으로 publish 된 메시지를 받는 곳이다
    @MessageMapping("/api/chat/message")
    public void message(@RequestBody ChatMessageRequestDto messageRequestDto, @Header("token") String token) {

        // 로그인 회원 정보를 들어온 메시지에 값 세팅
        User user = jwtTokenProvider.getAuthenticationUser(token);
        messageRequestDto.setUserId(user.getId());
        messageRequestDto.setSender(user.getUsername());

        // 메시지 생성 시간 삽입
        SimpleDateFormat sdf = new SimpleDateFormat("YYYY-MM-dd HH:mm");
        Calendar cal = Calendar.getInstance();
        Date date = cal.getTime();
        sdf.setTimeZone(TimeZone.getTimeZone("Asia/Seoul"));
        String dateResult = sdf.format(date);
        messageRequestDto.setCreatedAt(dateResult);

        // DTO 로 채팅 메시지 객체 생성
        ChatMessage chatMessage = new ChatMessage(messageRequestDto, userService);

        // 웹소켓 통신으로 채팅방 토픽 구독자들에게 메시지 보내기
        chatMessageService.sendChatMessage(chatMessage);

        // MySql DB에 채팅 메시지 저장
        chatMessageService.save(chatMessage);
    }
```
- ```StompHandler```를 통과한 채팅메시지 정보는 ```ChatMessageController```에 도착하게 됩니다.
- 일반적인 CRUD에서 사용하는 Mapping 어노테이션과는 다르게 ```@MessageMapping```으로 처리됩니다.
- 헤더에 담긴 토큰에서 정보를 꺼내 메시지를 보낸 사용자를 맵핑합니다.
- 메시지를 채팅방 토픽 구독자들에게 보낸 후 DB 저장이 일어나기 때문에 메시지 생성 시간을 DB 저장 시점이 아닌 Controller에 메시지가 오고 
  다시 클라이언트로 보내지기 전에 찍어야 하기 때문에 Controller에서 처리하였습니다.
- 이후 ```ChatMessageService``` 내 클래스에 구현된 메서드들을 활용해 웹소켓 통신으로 구독자들에게 메시지를 발송하고, 뒤이어 MySql DB에 메시지 내용들을 저장합니다.

### 3. ChatMessageService
```java
    // 채팅방에 메시지 발송
    public void sendChatMessage(ChatMessage chatMessage) {
        if (ChatMessage.MessageType.ENTER.equals(chatMessage.getType())) {
            chatMessage.setMessage(chatMessage.getSender() + "님이 방에 입장했습니다.");
            chatMessage.setSender("[알림]");
        } else if (ChatMessage.MessageType.QUIT.equals(chatMessage.getType())) {
            chatMessage.setMessage(chatMessage.getSender() + "님이 방에서 나갔습니다.");
            chatMessage.setSender("[알림]");
        }
        redisTemplate.convertAndSend(channelTopic.getTopic(), chatMessage);
    }

    // ChatMessage DB 테이블에 채팅 메시지 내용 저장
    public void save(ChatMessage chatMessage) {
        ChatMessage message = new ChatMessage();
        message.setType(chatMessage.getType());
        message.setRoomId(chatMessage.getRoomId());
        message.setUser(userService.findById(chatMessage.getUserId()));
        message.setUserId(chatMessage.getUserId());
        message.setSender(chatMessage.getSender());
        message.setMessage(chatMessage.getMessage());
        message.setCreatedAt(chatMessage.getCreatedAt());
        chatMessageRepository.save(message);
    }
```
- ```ChatMessageService```에 구현된 두 개의 주요 메서드입니다.
- ```sendChatMessage``` 메서드는 들어오는 메시지를 TYPE 별로 구분해 입퇴장 메시지 내용을 추가합니다.
- redisTemplate에 구현된 ```convertAndSend``` 메서드를 활용하여 해당 메시지가 SEND된 채팅방 구독자들에게 메시지를 발송합니다.


</br>

회원 가입 기능
----------------

### UserSignupRequestDto
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class UserSignupRequestDto {

    // 한글 영어 - _ 숫자 로 변경 필요
    @NotNull
    @NotBlank(message = "닉네임 입력은 필수입니다.")
    @Pattern(regexp = "^[a-zA-Z0-9ㄱ-ㅎ가-힣-_/]{3,10}$", message = "3~10자리의 '-','_', 한글, 알파벳만 사용 가능합니다.")
    private String username;

    @NotNull
    @NotBlank(message = "비밀번호 입력은 필수입니다.")
    private String password;

    @NotNull
    @NotBlank(message = "이메일 입력은 필수입니다.")
    @Email(message = "이메일 형식으로 입력해 주세요.")
    private String email;

}
```
- @Pattern : 정규식을 이용 하여 3 ~ 10 자리의 '알파벳, 숫자, 한글, -, _, /' 이외 입력 시 Exception 처리
- @NotNull : 받아온 JSON 내용에 해당하는 key가 없을 경우 Exception 처리
- @NotBlank : 빈값, 공백인 경우 Exception 처리
- @Email : 이메일 형식이 아닌 경우 Exception 처리
- @Valid : Controller단에서 위와같은 Exception 처리가 있는지 없는지 확인

### UserController
```java
    //회원 가입시 이메일 중복체크
    @PostMapping("/api/user/signup/emailCheck")
    public Object validCheckEmail(@RequestBody Map<String, Object> param) {
        try {
            userService.signupValidCheck(param.get("email").toString());
            CustomMessageResponse customMessageResponse = new CustomMessageResponse("사용 가능한 Email입니다.",HttpStatus.OK.value());
            return customMessageResponse.SendResponse();
        } catch (Exception ignore) {
            CustomMessageResponse customMessageResponse = new CustomMessageResponse(ignore.getMessage(),HttpStatus.BAD_REQUEST.value());
            return customMessageResponse.SendResponse();
        }
    }
```

### UserService
```java
    public void signupValidCheck(String Email){
        if (userRepository.findByEmail(Email).isPresent()) {
            throw new IllegalArgumentException("해당 이메일은 이미 가입된 회원이 있습니다.");
        }
    }
```

- /api/user/signup/emailCheck : 중복 확인 버튼 클릭 시 발동 하며 Service의 signupValidCheck함수를 통해 이메일 중복 검사를 진행합니다.


비밀번호 찾기 기능
----------------
### UserController
```java
    //비밀번호 찾기
    @PostMapping("/api/user/findPassword")
    public Object findPasswordByEamil(@RequestBody Map<String, Object> param){
        try {
            int CertificationNumber = userService.findPasswordByEamil(param.get("email").toString());
            JsonObject jsonObj = new JsonObject();
            jsonObj.addProperty("CertificationNumber", Integer.toString(CertificationNumber));
            return ResponseEntity.ok().body(jsonObj.toString());
        } catch (Exception ignore) {
            CustomMessageResponse customMessageResponse = new CustomMessageResponse(ignore.getMessage(),HttpStatus.BAD_REQUEST.value());
            return customMessageResponse.SendResponse();
        }
    }

    // 비밀번호 변경
    @PutMapping("/api/user/changePassword")
    public Long updateUserPassword(@RequestBody Map<String, Object> param) {
            String email = param.get("email").toString();
            String password = param.get("password").toString();
            return userService.updateUserPassword(email,password);
            }
```

### UserService
```java
    public int findPasswordByEamil(String email) {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new IllegalArgumentException("가입되지 않은 E-MAIL 입니다."));
        try {
            return mailUtil.sendMail(user);
        } catch (Exception e) {
            return 0;
        }
    }

    @Transactional
    public Long updateUserPassword(String email, String password) {
            User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new IllegalArgumentException("가입되지 않은 E-MAIL 입니다."));
            user.setPassword(passwordEncoder.encode(password));
            return user.getId();
            }
```

### MailUtil
```java
@Component
//메일을 보낼 클래스
public class MailUtil {
    @Value("${spring.mail.host}")
    private String hostSMTP;

    @Value("${spring.mail.username}")
    private String hostSMTPid;

    @Value("${spring.mail.password}")
    private String hostSMTPpw;

    public int sendMail(User user) throws Exception {
        Random r = new Random();
        int dice = r.nextInt(157211)+48271;

        //Mail Server 설정
        String charSet = "utf-8";
        //보내는 사람
        String fromEmail = this.hostSMTPid + "@naver.com";
        String fromName = "gaemangtalk";

        String title = "비밀번호 찾기 인증 이메일 입니다.";    //제목
        String content =

                System.getProperty("line.separator")+

                        System.getProperty("line.separator")+

                        "안녕하세요 회원님 저희 홈페이지를 찾아주셔서 감사합니다"

                        +System.getProperty("line.separator")+

                        System.getProperty("line.separator")+

                        "비밀번호 찾기 인증번호는 " +dice+ " 입니다. "

                        +System.getProperty("line.separator")+

                        System.getProperty("line.separator")+

                        "받으신 인증번호를 홈페이지에 입력해 주시면 다음으로 넘어갑니다."; // 내용

        //email 전송
        String mailRecipient = user.getEmail(); //받는 사람 이메일 주소
        try {
            //객체 선언
            HtmlEmail mail = new HtmlEmail();
            mail.setDebug(true);
            mail.setCharset(charSet);
            mail.setSSLOnConnect(true); //SSL 사용
            mail.setHostName(hostSMTP);
            mail.setSmtpPort(465); //SMTP 포트 번호
            mail.setAuthentication(hostSMTPid, hostSMTPpw);
            mail.setStartTLSEnabled(true); //TLS 사용
            mail.addTo(mailRecipient, charSet);
            mail.setFrom(fromEmail, fromName, charSet);
            mail.setSubject(title);
            mail.setHtmlMsg(content);
            mail.send();

            return dice;
        }catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }
}
```

- /api/user/findPassword - userService.findPasswordByEamil에서 가입된 이메일이 있을경우 MailUtil이 동작합니다.
  MailUtil에서는 랜덤 함수를 통해 인증번호를 생성하고 미리SMTP허용한 이메일 정보를 통해 메일 발송 후 Client에 인증번호 발송하여 비교할 수 있도록 합니다.
- /api/user/changePassword - userService.updateUserPassword를 통해 변경 할 비밀번호변경

</br>

로그인 기능
--------------------

### JwtTokenProvider
```java
@Component
public class JwtTokenProvider {

    private String secretKey = "webfirewood";

    // 토큰 유효시간 30분
    private long tokenValidTime = 30 * 60 * 1000L;

    private final UserDetailsServiceImpl userDetailsServiceImpl;

    @Autowired
    public JwtTokenProvider(UserDetailsServiceImpl userDetailsServiceImpl) {
        this.userDetailsServiceImpl = userDetailsServiceImpl;
    }

    // 객체 초기화, secretKey를 Base64로 인코딩한다.
    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
    }

    // JWT 토큰 생성
    public String createToken(Long userPk) {
        String stringUserPk = userPk.toString();
        Claims claims = Jwts.claims().setSubject(stringUserPk); // JWT payload 에 저장되는 정보단위
        Date now = new Date();
        return Jwts.builder()
                .setClaims(claims) // 정보 저장
                .setIssuedAt(now) // 토큰 발행 시간 정보
                .setExpiration(new Date(now.getTime() + tokenValidTime)) // set Expire Time
                .signWith(SignatureAlgorithm.HS256, secretKey)  // 사용할 암호화 알고리즘과
                // signature 에 들어갈 secret값 세팅
                .compact();
    }

    // JWT 토큰에서 인증 정보 조회
    public Authentication getAuthentication(String token) {
        UserDetails userDetails = userDetailsServiceImpl.loadUserById(Long.parseLong(this.getUserPk(token)));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    // JWT 토큰에서 User 가져오기
    public User getAuthenticationUser(String token){
        UserDetailsImpl userDetails = userDetailsServiceImpl.loadUserById(Long.parseLong(this.getUserPk(token)));
        return userDetails.getUser();
    }

    // JWT 토큰에서 username 조회
    public String getAuthenticationUsername(String token) {
        UserDetails userDetails = userDetailsServiceImpl.loadUserById(Long.parseLong(this.getUserPk(token)));
        return userDetails.getUsername();
    }

    // 토큰에서 회원 정보 추출
    public String getUserPk(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    }

    // Request의 Header에서 token 값을 가져옵니다
    public String resolveToken(HttpServletRequest request) {
        return request.getHeader("token");
    }

    // 토큰의 유효성 + 만료일자 확인
    public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwtToken);
            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
}
```

### UserController
```java
    //로그인
    @PostMapping("/api/user/login")
    public Object loginUser(@Valid @RequestBody UserLoginRequestDto userLoginRequestDto) {
        try {
            User user = userService.loginValidCheck(userLoginRequestDto);
            JsonObject jsonObj = new JsonObject();
            jsonObj.addProperty("token", jwtTokenProvider.createToken(user.getId()));
            return ResponseEntity.ok().body(jsonObj.toString());
        } catch (Exception ignore) {
            CustomMessageResponse customMessageResponse = new CustomMessageResponse(ignore.getMessage(),HttpStatus.BAD_REQUEST.value());
            return customMessageResponse.SendResponse();
        }
    }

    // 카카오 로그인
    // 프론트엔드에서 처리 후 카카오 토큰을 백으로 넘겨 주어 JWT token, username, userid 반환
    @PostMapping("/api/user/kakaoLogin")
    public Object loginUser(@RequestBody Map<String, Object> param) {
            try {
            JsonObject jsonObj = userService.kakaoLogin(param.get("kakaoToken").toString());
            return ResponseEntity.ok().body(jsonObj.toString());
            } catch (Exception ignore) {
            CustomMessageResponse customMessageResponse = new CustomMessageResponse(ignore.getMessage(),HttpStatus.BAD_REQUEST.value());
            return customMessageResponse.SendResponse();
            }
        }
```

### UserService
```java
    public User loginValidCheck(UserLoginRequestDto userLoginRequestDto){
        User user = userRepository.findByEmail(userLoginRequestDto.getEmail())
                .orElseThrow(() -> new IllegalArgumentException("가입되지 않은 E-MAIL 입니다."));
        if (!passwordEncoder.matches(userLoginRequestDto.getPassword(), user.getPassword())) {
            throw new IllegalArgumentException("잘못된 비밀번호입니다.");
        }
        return user;
    }

    public JsonObject kakaoLogin(String accessToken) {
            // 카카오 OAuth2 를 통해 카카오 사용자 정보 조회
            KakaoUserInfo userInfo = kakaoOAuth2.getUserInfo(accessToken);
            Long kakaoId = userInfo.getId();
            String nickname = userInfo.getNickname();
            String email = userInfo.getEmail();
    
            // DB 에 중복된 Kakao Id 가 있는지 확인
            User kakaoUser = userRepository.findByKakaoId(kakaoId).orElse(null);
    
            // 카카오 정보로 회원가입
            if (kakaoUser == null) {
            // 카카오 이메일과 동일한 이메일을 가진 회원이 있는지 확인
            User sameEmailUser = null;
    
            if (email != null) {
            sameEmailUser = userRepository.findByEmail(email).orElse(null);
            }
            if (sameEmailUser != null) {
            kakaoUser = sameEmailUser;
            // 카카오 이메일과 동일한 이메일 회원이 있는 경우
            // 카카오 Id 를 회원정보에 저장
            kakaoUser.setKakaoId(kakaoId);
            userRepository.save(kakaoUser);
            } else {
                // 카카오 정보로 회원가입
                // username = 카카오 nickname
                String username = nickname;
                // password = 카카오 Id + ADMIN TOKEN
                String password = kakaoId + ADMIN_TOKEN;
                // 패스워드 인코딩
                String encodedPassword = passwordEncoder.encode(password);
                // ROLE = 사용자
                UserRole role = UserRole.USER;
        
                if (email != null) {
                kakaoUser = new User(username, encodedPassword, email, role, kakaoId);
                } else {
                kakaoUser = new User(username, encodedPassword, role, kakaoId);
                }
                userRepository.save(kakaoUser);
                }
            }
    
            // 스프링 시큐리티 통해 로그인 처리
            UserDetailsImpl userDetails = new UserDetailsImpl(kakaoUser);
            Authentication authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authentication);
            JsonObject jsonObj = new JsonObject();
            jsonObj.addProperty("token", jwtTokenProvider.createToken(userDetails.getUser().getId()));
            return jsonObj;
        }
```
- /api/user/login : userService.loginValidCheck을 통해 현재 회원가입이 되어있는 사용자가 맞는지 비밀번호는 일치하는지 비교 후 jwtTokenProvider.createToken(user.getId()) 을 통해 Server에서 관리하는 Token을 생성 후 ResponseBody에 실어 보내줍니다.
- /api/user/kakaoLogin : Client에서 kakao로그인을 하여 발급받은 kakaoToken을 받아 kakaoId, nickname, Email을 통해 db에 저장합니다. 만약 db에 일치하는 Email이 있을 경우 kakaoId만 업데이트 시켜줍니다.
  이후 Server에서 현재 로그인 상태를 확인 할 수 있도록 Server에서 관리 하는 Token을 새로 생성하여 ResponseBody에 실어 보내줍니다.
  </br>

보안관련 기능
--------------------
### WebSecurityConfig
```java
@Configuration
@EnableWebSecurity // 스프링 Security 지원을 가능하게 함
@EnableGlobalMethodSecurity(securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    private final JwtTokenProvider jwtTokenProvider;

    @Autowired
    public WebSecurityConfig (JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Bean
    public BCryptPasswordEncoder encodePassword() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.headers().frameOptions().disable();
        http.authorizeRequests()
                // login 없이 허용
                .antMatchers("/chatting/**").permitAll()
                .antMatchers("/api/user/login").permitAll()
                .antMatchers("/api/user/kakaoLogin").permitAll()
                .antMatchers("/api/user/signup").permitAll()
                .antMatchers("/api/user/signup/emailCheck").permitAll()
                .antMatchers("/api/user/findPassword").permitAll()
                .antMatchers("/api/user/changePassword").permitAll()
                .antMatchers("/api/chat/message").permitAll()

                // 그 외 모든 요청은 인증과정 필요
                .anyRequest().authenticated()
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                        UsernamePasswordAuthenticationFilter.class);

        // JwtAuthenticationFilter를 UsernamePasswordAuthenticationFilter 전에 넣는다
    }
}
```

### JwtAuthenticationFilter
```java
public class JwtAuthenticationFilter extends GenericFilterBean {

    private final JwtTokenProvider jwtTokenProvider;

    @Autowired
    public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 헤더에서 JWT 를 받아옵니다.
        String token = jwtTokenProvider.resolveToken((HttpServletRequest) request);
        // 유효한 토큰인지 확인합니다.
        if (token != null && jwtTokenProvider.validateToken(token)) {
            // 토큰이 유효하면 토큰으로부터 유저 정보를 받아옵니다.
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            // SecurityContext 에 Authentication 객체를 저장합니다.
            SecurityContextHolder.getContext().setAuthentication(authentication);
        } else {
            if (((HttpServletRequest) request).getMethod().toString().equals("OPTIONS")) {
                ((HttpServletResponse) response).addHeader("Access-Control-Allow-Origin", "*");
                ((HttpServletResponse) response).addHeader("Access-Control-Allow-Methods", "*");
                ((HttpServletResponse) response).addHeader("Access-Control-Allow-Headers", "*");
                return;
            }
            SecurityContextHolder.getContext().setAuthentication(null);
        }
        chain.doFilter(request, response);
    }
}
```
- WebSecurityConfig.configure안에 .addFilterBefore을 통해 Security가 동작 하기 전  JwtAuthenticationFilter에 을 먼저 실행합니다.
- JwtAuthenticationFilter.doFilter를 Override 하여 Server에서 생성 한 Token이 헤더에 실려 올 경우 Authentication을 Context에 넣어 주었습니다.
- 헤더에 Token이 실려오지 않을 경우 Context에 있는 Authentication로 변경 해 주었으며 CORS 호출 시에는 GET, POST 등 요청할 경우 OPTIONS을 먼저 보내 가도 되는곳인지 확인하는 과정이 있어 OPTIONS을 분기처리 하였습니다.


</br>

마이 페이지 관련 기능
-----------------

### S3Uploader
```java
    @Slf4j // 로깅을 위한 어노테이션
    @Component // 빈 등록을 위한 어노테이션
    @RequiredArgsConstructor // final 변수에 대한 의존성을 추가합니다.
    public class S3Uploader implements Uploader {
    
        private final AmazonS3Client amazonS3Client;
    
        @Value("${cloud.aws.s3.bucket}")  // 프로퍼티에서 cloude.aws.s3.bucket에 대한 정보를 불러옵니다.
        public String bucket;
    
        public String upload(MultipartFile multipartFile, String dirName) throws IOException {
            File convertedFile = convert(multipartFile);
            return upload(convertedFile, dirName);
        }
    
        private String upload(File uploadFile, String dirName) {
            String fileName = dirName + "/" + uploadFile.getName();
            String uploadImageUrl = putS3(uploadFile, fileName);
            removeNewFile(uploadFile);
            return uploadImageUrl;
        }
    
        private String putS3(File uploadFile, String fileName) {
            amazonS3Client.putObject(new PutObjectRequest(bucket, fileName, uploadFile).withCannedAcl(CannedAccessControlList.PublicRead));
            return amazonS3Client.getUrl(bucket, fileName).toString();
        }
    
        private void removeNewFile(File targetFile) {
            if (targetFile.delete()) {
                return;
            }
            log.info("임시 파일이 삭제 되지 못했습니다. 파일 이름: {}", targetFile.getName());
        }
    
        private File convert(MultipartFile file) throws IOException {
            File convertFile = new File(file.getOriginalFilename());
            if (convertFile.createNewFile()) {
                try (FileOutputStream fos = new FileOutputStream(convertFile)) {
                    fos.write(file.getBytes());
                }
                return convertFile;
            }
            throw new IllegalArgumentException(String.format("파일 변환이 실패했습니다. 파일 이름: %s", file.getName()));
        }
    
    }
```
- 클라이언트로부터 받은 파일데이터를 S3 이미지 스토리지에 업로드하고, 받은 파일 데이터를 삭제안 뒤 URL만을 반환해주는 Util Class 입니다.

### S3 Controller
```java
    @PostMapping("/api/s3upload")
    public String imgUpload(@RequestParam("data") MultipartFile file) throws IOException {
        String profileUrl = uploader.upload(file, "static");
        return profileUrl;
    }
```
- 마이페이지 프로필을 수정할 때 바로 파일을 받아서 DB에 넣어도 되지만, 이렇게 POST 메서드를 따로 만든 이유는 재사용성 때문입니다.
- 채팅방 대표이미지 업로드 등 다른 기능에서도 해당 URL을 사용해 이미지 URL을 반환받은 후 JSON 형태로 POST, PUT 요청을 다시 보냅니다.
이럴 경우, 백엔드 입장에서는 여러 Controller에서 Parameter 마다 파일을 받는다고 설정해주지 않아도 되어 좋습니다. 
  프론트엔드 입장에서도 미리보기 등을 구현할 수 있어 기능성 측면에서 좋습니다.

### UserController
```java
    // 마이페이지 프로필 조회
    // token 키 값으로 Header 에 실어주시면 된다!!
    @GetMapping("/api/user/profile")
    public User getMyProfile(@AuthenticationPrincipal UserDetailsImpl userDetails) {
        return userService.findById(userDetails.getUser().getId());
    }

    // 마이페이지 프로필 수정
    // username, email, profileurl 만 바꿀 수 있도록 함
    @PutMapping("api/user/profile/{userId}")
    public Object updateMyProfile(@PathVariable Long userId, @Valid @RequestBody UserProfileRequestDto userProfileRequestDto) {
        try {
            return userService.myProfileUpdate(userId, userProfileRequestDto);
        } catch (Exception ignore) {
            CustomMessageResponse customMessageResponse = new CustomMessageResponse(ignore.getMessage(),HttpStatus.BAD_REQUEST.value());
            return customMessageResponse.SendResponse();
        }
    }
```
- 다음은 프로필 조회, 수정 기능을 담고 있는 UserController의 일부입니다.