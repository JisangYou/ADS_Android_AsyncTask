# ADS04 Android

## 수업 내용

- AsyncTask를 학습

## Code Review

### MainActivity

```Java
/**
 * AsyncTask = 세 개의 기본함수를 지원하는 Thread
 * 1. onPreExecute = doInBackground ()함수가 실행되기 전에 실행되는 함수
 * 2. doInBackground : 백그라운(sub thread)에서 코드를 실행하는 함수 <에만 sub thread
 *         v onPostExecute는 doInBackground로 부터 데이터를 받을 수 있다.
 * 3. onPostExecute : doInBackground()함수가 실행된 후에 실행되는 함수
 *
 */

public class MainActivity extends AppCompatActivity {

    TextView textView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView)findViewById(R.id.textView);

        getServer("http://google.com");
    }

    private void getServer(String url) {
       AsyncTask<String, Void, String> task =  new AsyncTask<String, Void, String>(){
           //1. doInBackground 함수의 파라미터로 사용
           //2. onProgressUpdate 함수의 파라미터로 사용
           //   주로 진행상태의 percent 값(int)으로 사용된다.
           //3. doInBackground의 리턴값이면서 onPostExeCute 파라미터

           @Override
            protected String doInBackground(String... args) {
                String param1 = args[0];
//                Remote remote = new Remote(); // 클래스를 선언해버리면, 이중으로 메모리가 올라감.
                String result = Remote.getData(param1); // Remote클래스가 static으로 선언되어 있기때문에, 따로 객체를 생성해줄 필요가 없다.

                return result;
            }

           @Override
           protected void onPostExecute(String result) {
               // 전체 html 코드 중에서
               //<title></title>안에 있는 내용만 출력하세요
              String title = result.substring(result.indexOf("<title>")+"<title>".length(), result.indexOf("</title>"));
               textView.setText(title);
//               super.onPostExecute(result);
           }
       };
        task.execute(url);
    }
}
```

### Remote

```Java
public class Remote {

    public static String getData(String string){
        final StringBuilder result = new StringBuilder();
        try {
            URL url = new URL(string);
            HttpURLConnection con = (HttpURLConnection)url.openConnection();
            con.setRequestMethod("GET");
            // 통신이 성공인지 체크
            if (con.getResponseCode() == HttpURLConnection.HTTP_OK) {
                // 여기서 부터는 파일에서 데이터를 가져오는 것과 동일
                InputStreamReader isr = new InputStreamReader(con.getInputStream());
                BufferedReader br = new BufferedReader(isr);
                String temp = "";
                while ((temp = br.readLine()) != null) {
                    result.append(temp).append("\n");
                }
                br.close();
                isr.close();
            } else {
                Log.e("ServerError", con.getResponseCode()+"");
            }
            con.disconnect();
        }catch(Exception e){
            Log.e("Error", e.toString());
        }
        return result.toString();
    }

}

```
- Remote라는 클래스를 static으로 선언함으로써 어느 클래스건 통신을 하려고 할때 따로 객체를 생성해주지 않고도 사용가능
- 통신을 하는 클래스로써 관련 내용은 [networkBasic](https://github.com/youjisang/ADS_Android_Network.git) 참고

## 보충설명

안드로이드에서 UI를 조작할 수 있는 방법 3가지

1. Handler와 Looper 
2. runOnUiThread( )
3. AsyncTask

### AsyncTask란?

- AsyncTask는 완성되어 있지 않은 상태(abstract)로 안드로이드에 포함되어 있는 추상클래스
- AsyncTask는 UI 처리 및 Background 작업 등을 하나의 클래스에서 작업 할 수 있게 지원
- 즉, 메인Thread와 일반Thread를 가지고 Handler를 사용하여 핸들링하지 않아도 AsyncTask 객체하나로 각각의 주기마다 CallBack 메서드가 편하게 UI를 수정 할 수 있고, Background 작업을 진행 할 수 있음.


### 동작 순서

![AsyncTask](http://cfile23.uf.tistory.com/image/2420B240577D4A720F8136)

 1. execute( ) 명령어를 통해 AsyncTask을 실행

 2. AsyncTask로 백그라운드 작업을 실행하기 전에 onPreExcuted( )실행. 이 부분에는 이미지 로딩 작업이라면 로딩 중 이미지를 띄워 놓기 등, 스레드 작업 이전에 수행할 동작을 구현

 3. 새로 만든 스레드에서 백그라운드 작업을 수행, execute( ) 메소드를 호출할 때 사용된 파라미터를  전달 받음

 4. doInBackground( ) 에서 중간 중간 진행 상태를 UI에 업데이트 하도록 하려면 publishProgress( ) 메소드를 호출

 5. onProgressUpdate( ) 메소드는 publishProgress( )가 호출 될 때마다 자동으로 호출

 6. doInBackground( ) 메소드에서 작업이 끝나면 onPostExcuted( ) 로 결과 파라미터를 리턴하면서 그 리턴값을 통해 스레드 작업이 끝났을 때의 동작을 구현

※ onPreExecute( ), onProgressUpadate( ), onPostExecute( ) 메소드는 메인 스레드에서 실행되므로 UI 객체에 자유롭게 접근



### 사용 예제코드

![asynctask](https://4.bp.blogspot.com/-K_DUtYI4XQg/V4fUJRWtN6I/AAAAAAAALl8/TK9Izfszt8k_mqBJfEecqx1dcZbhqUhWQCLcB/w1200-h630-p-k-no-nu/Untitled.png)

- AsyncTask<doInBackground()의 변수 종류, onProgressUpdate()에서 사용할 변수 종류, onPostExecute()에서 사용할 변수종류>

```Java

doInBackground()의 변수 종류 : 우리가 정의한 AsyncTask를 execute할 때 전해줄 값의 종류
onProgressUpdate()에서 사용할 변수 종류 : 진행상황을 업데이트할 때 전달할 값의 종류
onPostExecute()에서 사용할 변수종류 : AsyncTask가 끝난 뒤 결과값의 종류

```
- Void 타입 사용 시에는 간단하게 작성할 수 있음.

```Java
private class DownloadFilesTask extends AsyncTask<Void, Void, Void> { ... }
```

- AsyncTask는 execute(Runnable runnable) 메소드도 있어서 간단한 스레드 작업을 할 때도 사용

```Java
AsyncTask.execute(
    new Runnable() { 
        @Override 
        public void run() { // TODO Auto-generated method stub 
        // 스레드 작업 
        } 
    });
```
- ※ 하나의 객체로 재사용이 불가능함 



### 출처

- 출처: http://itmining.tistory.com/7 [IT 마이닝]
- 출처: http://corej21.tistory.com/38
- 출처: http://itmir.tistory.com/624 [미르의 IT 정복기]
- 출처: http://daehyub71.tistory.com/entry/안드로이드AsyncTask에-대하여 [대협의 일상]
- 출처: http://corej21.tistory.com/38 [인생, 스레드!]

## TODO

- Asynctask의 콜백구조 복습
- 내부적으로 어떻게 작동하는지 연구
- 단점연구(하나의 객체로 재사용이 불가능하다는 것이 어떤 상황에서 불편한건지, 메모리나 성능 측면에서는 어떤지 등)

## Retrospect

- 핸들러와 쓰레드를 함께 사용하는 것보다 사용하기 훨씬 편한 것 같다. 그러나 단점이나 제약도 있으니 이점 찾아보고 유의할 것.

## Output
- 생략

