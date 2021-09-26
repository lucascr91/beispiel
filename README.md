## Beispiel

Este repositório apresenta um exemplo de implementação técnica de tagueamento em Android usando Firebase. Para o exemplo, foi usado um aplicativo desenvolvido no curso [Android Java Masterclass - Become an App Developer](https://www.udemy.com/course/master-android-7-nougat-java-app-development-step-by-step/) de Tim Buchalka. O aplicativo, chamado "Top 10 downloaded", usa o rss feed da loja da Apple para apresentar feeds atualizados com os items mais baixados da loja de acordo com a categoria definida pelo usuário (free apps, paid apps, etc.). Abaixo são apresentados screenshots que dão uma ideia do funcionamento do app:

  Top Paid Apps            |  Top Free Apps
:-------------------------:|:-------------------------:
![](https://github.com/lucascr91/beispiel/blob/master/images/topPaid.png)  |  ![](https://github.com/lucascr91/beispiel/blob/master/images/topFree.png)

### Configuração Firebase:

Uma descrição completa de como configurar o firebase para Android pode ser encontrada na documentação oficial: https://firebase.google.com/docs/android/setup?authuser=0 . Aqui apenas sumarizo os principais passos:

1. Cria um projeto no firebase
2. Registre o app
3. Adiciona o arquivo de configuração `google-services.json` [^1]
4. Adiciona a SDK do firebase no app

**Observação:** Diferentemente do que diz a documentação atualmente (26/09/2021), o plugin do firebase deve ser instalado usando `id` e não `apply plugin` (veja essa questão no [stackoverflow](https://stackoverflow.com/questions/64538836/can-not-link-connect-android-studio-with-firebase)):

```
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // Google Services plugin
}
```
### Mandando eventos para o firebase:

Após a configuração, o firebase já recebe por default uma série de eventos, como screen view, user engagement, etc. Vamos adicionar um evento personalizado que dispara quando o usuário dispara o aplicativo. Para isso, vamos criar uma variável do tipo `FirebaseAnalytics` e usar ela dentro do método `OnCreate`. No snippet abaixo, foram adicionadas as linhas 3, 15-18 em relação ao código original do app.

```java
public class MainActivity extends AppCompatActivity {

    private FirebaseAnalytics firebaseAnalytics;
    private static final String TAG = "MainActivity";
    private ListView listApps;
    private String feedUrl="http://ax.itunes.apple.com/WebObjects/MZStoreServices.woa/ws/RSS/topfreeapplications/limit=%d/xml";
    private int feedLimit = 10;
    private String feedCacheUrl =  "INVALIDATED";
    public static final String STATE_URL = "feedUrl";
    public static final String STATE_LIMIT = "feedLimit";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        firebaseAnalytics = FirebaseAnalytics.getInstance(this);
        Bundle bundle = new Bundle();
        bundle.putString(FirebaseAnalytics.Param.ITEM_ID, "12345");
        firebaseAnalytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, bundle);

        setContentView(R.layout.activity_main);
        listApps = (ListView) findViewById(R.id.xmlListView);

        if (savedInstanceState != null) {
            feedUrl = savedInstanceState.getString(STATE_URL);
            feedLimit = savedInstanceState.getInt(STATE_LIMIT);
        }

        downloadUrl(String.format(feedUrl, feedLimit));
    }
```

Após a alteração no código, é possível checar no console do firebase que os dados estão sendo disparados, como mostra a figura abaixo:

![](https://github.com/lucascr91/beispiel/blob/master/images/basicEventsFirebase.png)

Alternativamente, é possível analisar os disparos em tempo real na "Debug View". Para apontar os disparos para o Debug View, você precisa primeiro registrar seu dispositivo virtual como um dispositivo de depuração válido. Para fazer isso, abra o terminal dentro do Android Studio com o dispositivo virtual em execução e digite:

```
adb shell setprop debug.firebase.analytics.app com.example.top10downloaded
```

(Se não tiver o adb instalado, baixe [aqui](https://developer.android.com/studio/releases/platform-tools) e coloque ele em seu path)

Agora os eventos disparados durante a sessão do usuário aparecem no debug view:

![](https://github.com/lucascr91/beispiel/blob/master/images/basicEventsFirebaseDebugView.png)