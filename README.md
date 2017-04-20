				
					-------------------- Today's Topic ---------------------
									(1) Notification
					--------------------------------------------------------


 - 프로젝트 명 : [MediaPlayerService]

 - 내용 : 사용자가 버튼을 누르면 상태바에 알림창을 띄우게 합니다.[MediaPlayerService] 
									

#1. Notification - [MediaPlayerService]

## 화면 출력

서비스를 구현하는 어플리케이션에서 많이 사용되는 기능입니다. 카카오톡에서 메시지가 왔을 때, 핸드폰 상단에 알림창이 뜨는 거처럼 화면 상단에 알림이 뜨게 합니다.  


![](http://i.imgur.com/nHS4Rfr.png)

버튼을 클릭하면 서비스가 실행되면서 화면 상단에 알림바가 생성됩니다.

![](http://i.imgur.com/IRVF50h.png)

이 프로젝트에서는 음악을 직접 실행하지는 않지만, 음악 실행과 관련된 알림들과 버튼 모양 등을 실제 사용에 가깝게 구현하였습니다. 

핵심 코드가 구현되어 있는 "MediaPlayerService" 위주로 설명 드리겠습니다.

![](http://i.imgur.com/EeREhDb.png)


##1.1 onStartCommand()

[MediaPlayerService]

![](http://i.imgur.com/pR4MzoP.png)

--> MainActivity에서 처음 서비스를 실행시켰을 때 호출되는 함수입니다.

(1) mManager는 MediaSession(미디어 재생 기능을 지원하기 위한 클래스)의 참조변수입니다.

(2) initMediaSessions() : 미디어 관련 API들을 세팅합니다. 
 
(3) handleIntent() : //

##1.2 initMediaSessions()

[MediaPlayerService]

![](http://i.imgur.com/cI9jzTW.png)
![](http://i.imgur.com/T6KK84X.png)

(1) mMediaPlayer는 음악을 재생시키기 위한 control을 제공합니다. 객체 선언한 이후 음악이 내장된 path를 설정합니다. 준비 메서드인 prepare()를 호출합니다.
					
					mMediaPlayer = new MediaPlayer();
					mMediaPlayer.setDataSource(musicUri.getPath());
				    mMediaPlayer.prepare();

(2) MediaSession에서 세션을 만들고, 그 세션으로부터 토큰을 받아와 미디어 컨트롤(mController)에 설정합니다.


        mSession = new MediaSession(getApplicationContext(), "Media Player Session");
        mController =new MediaController(getApplicationContext(), mSession.getSessionToken());

(3) 세션은 음악의 각각의 상태(pause, play, stop 등)를 정의하기 위한 콜백함수를 부릅니다. 각 상태마다 알림 상태바가 달라야 하기 때문에 Notification이 새롭게 호출되어야 합니다. 
				
						mSession.setCallback()

(4) onStop() : 알림을 관리하는 NotificationManager를 얻어온 뒤, cancel() 메소드를 호출하여 Notification을 해제하면 상태바가 사라집니다. 여기서는 서비스 또한 같이 종료되게 하였습니다.


##1.3 buildNotification()

[MediaPlayerService]

![](http://i.imgur.com/pqjkTyb.png)
![](http://i.imgur.com/1p8dcUh.png)

--> 알림을 설정하고 띄우는 역할입니다.

(1) 노티바 모양을 결정합니다. // but, 이번 프로젝트에서는 사용하지 않았습니다.........

			Notification.MediaStyle style = new Notification.MediaStyle();

(2) 노티바 전체를 클릭했을 때, 다음에 실행되는 동작을 intent에 담아둡니다. 여기서는 음악을 멈추게 하기 위하여 ACTION_STOP을 설정하였습니다.

				Intent intent = new Intent( getApplicationContext(), MediaPlayerService.class );
    			intent.setAction( ACTION_STOP );

(3) PendingIntent는 인텐트의 발생을 지연시킵니다. 즉 노티바 전체를 클릭하여 음악을 멈추게 하는 동작을 지연시킬 목적으로 사용합니다. PendingIntent를 생성하는데 필요한 인자로서 노티바나 위젯을 사용하기 위해서는 시스템 자원이 필요합니다. 

    PendingIntent pendingIntent = PendingIntent.getService(getApplicationContext(), 1, intent, 0);
															(Context context, int requestCode, Intent intent, int flags)


(4) 노티바를 생성하는 곳입니다. 알림(Notification)을 만들어내는 Builder 객체를 생성하고 필요한 속성들을 설정합니다. 

   			 Notification.Builder builder = new Notification.Builder( this )
   			         .setSmallIcon(R.mipmap.ic_launcher) 					//상태표시줄에 보이는 아이콘 모양	
    		        .setContentTitle( "Media Title" )                       // 노래제목
    		        .setContentText( "Media Artist" )                       // 가수
      		        .setDeleteIntent( pendingIntent )                       // 슬라이드로 노티바를 지운다.
       		        .setStyle(style);										// 노티바 모양


(5) 노티바에 있는 각 버튼의 아이콘과 리스너를 설정합니다. 인자값으로 Action 타입을 주어야 합니다. 여기서는 generateAction() 사용자 정의 함수를 만들어서 Action 타입의 변수가 리턴됩니다.

				builder.addAction(Action action)

(6) OS 별로 액션의 개수가 정해져 있으므로 Action의 중요도에 따라 꼭 보여져야 되는 Action 을 앞쪽에 배치합니다. 번호는 순서대로 0번부터.....입니다. 이 프로젝트는 롤리팝 이하에서도 실행되어야 하기 때문에 1,2,3 번만 보여주도록 하였습니다. 그 이상 버전에서는 5개 모두 나옵니다ㅣ.

					style.setShowActionsInCompactView(1,2,3,0,4);

(7) NotificationManager를 선언하고 노티바를 화면에 보여줍니다. notify() 안에 있는 인자는 노티바의 아이디와 Notifiaction(builder 객체를 build()하면 생성됩니다.) 입니다. **즉 NotificationManager가 리스트에 등록되어있는 Notification을 화면에 띄워주는 작업니다.** 

				NotificationManager notificationManager = (NotificationManager) getSystemService( Context.NOTIFICATION_SERVICE );
        
        		notificationManager.notify( 1, builder.build() );

##1.4 generateAction()


[MediaPlayerService]

![](http://i.imgur.com/xFkZNEr.png)

--> Activity 에서 클릭 버튼을 생성합니다.

(1) intent 객체의 액션은 호출된 함수에서 넘어온 패러미터 값으로 설정합니다.( ACTION_PLAY, ACTION_PAUSE, ACTION_REWIND, ACTION_STOP 등) 

					Intent intent = new Intent( getApplicationContext(), MediaPlayerService.class );
					intent.setAction( intentAction );



(2) 1.3에서도 설명하였지만 PendingIntent는 실행 대상이 되는 인텐트를 지연시키는 역할입니다. 노티바에 있는 버튼들을 눌렀을 때, 인텐트 실행을 지연시킵니다.

					 PendingIntent pendingIntent = PendingIntent.getService(getApplicationContext(), 1, intent, 0);

(3) OS 버전별 Action Builder를 처리합니다. builder.addAction()에게 action 값을 반환하기 위해 build() 메소드를 호출해서 action 값을 생성, 반환합니다. 

    
  				  if ( Build.VERSION.SDK_INT >= Build.VERSION_CODES.M ) {
 			      		 Icon iconTemp = Icon.createWithResource(getBaseContext(),icon);
  				         return new Notification.Action.Builder(iconTemp, title, pendingIntent).build();
 				  } else {
  			      	     return new Notification.Action.Builder(icon, title, pendingIntent).build();
				  }

##1.5 handleIntent()


[MediaPlayerService]

![](http://i.imgur.com/4jf6CKw.png)

--> Intent Action 에 넘어온 명령어를 분기시키는 함수입니다. 각 action에 대한 처리들을 controller가 받아서 세션으로 넘기면 1.2 initMediaSessions()의 setCallback이 받아서 Notification 바를 생성하고 버튼 생성 등의 역할을 수행합니다.

