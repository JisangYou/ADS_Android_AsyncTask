# AsyncTask
## 개념 및 메소드

- AsyncTask는 handler와 thread 사용을 편리하게 사용하기 위해 만들어진 클래스
- 한 클래스 안에서 Ui작업과 비즈니스 로직 처리가 모두 가능하기 때문에 간편
- 하나의 객체로 재사용이 불가능함

```Java
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
