# Acceso-raiz
public class SplashActivity extends Activity {

    static {
        // Set settings before the main shell can be created
        Shell.enableVerboseLogging = BuildConfig.DEBUG;
        Shell.setDefaultBuilder(Shell.Builder.create()
            .setFlags(Shell.FLAG_REDIRECT_STDERR)
            .setTimeout(10)
        );
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Preheat the main root shell in the splash screen
        // so the app can use it afterwards without interrupting
        // application flow (e.g. root permission prompt)
        Shell.getShell(shell -> {
            // The main shell is now constructed and cached
            // Exit splash screen and enter main activity
            Intent intent = new Intent(this, MainActivity.class);
            startActivity(intent);
            finish();
        });
    }
}Shell.Result result;
// Execute commands synchronously
result = Shell.cmd("find /dev/block -iname boot").exec();
// Aside from commands, you can also load scripts from InputStream.
// This is NOT like executing a script like "sh script.sh", but rather
// more similar to sourcing the script (". script.sh").
result = Shell.cmd(getResources().openRawResource(R.raw.script)).exec();

List<String> out = result.getOut();  // stdout
int code = result.getCode();         // return code of the last command
boolean ok = result.isSuccess();     // return code == 0?

// Async APIs
Shell.cmd("setenforce 0").submit();   // submit and don't care results
Shell.cmd("sleep 5", "echo hello").submit(result -> updateUI(result));

// Run tasks and output to specific Lists
List<String> mmaps = new ArrayList<>();
Shell.cmd("cat /proc/1/maps").to(mmaps).exec();
List<String> stdout = new ArrayList<>();
List<String> stderr = new ArrayList<>();
Shell.cmd("echo hello", "echo hello >&2").to(stdout, stderr).exec();

// Receive output in real-time
List<String> callbackList = new CallbackList<String>() {
    @Override
    public void onAddElement(String s) { updateUI(s); }
};
Shell.cmd("for i in $(seq 5); do echo $i; sleep 1; done")
    .to(callbackList)
    .submit(result -> updateUI(result));public class ExampleInitializer extends Shell.Initializer {
    @Override
    public boolean onInit(Context context, Shell shell) {
        InputStream bashrc = context.getResources().openRawResource(R.raw.bashrc);
        // Here we use Shell instance APIs instead of Shell.cmd(...) static methods
        shell.newJob()
            .add(bashrc)                  /* Load a script */
            .add("export ENV_VAR=VALUE")  /* Run some commands */
            .exec();
        return true;  // Return false to indicate initialization failed
    }
}
Shell.Builder builder = /* Create a shell builder */ ;
builder.setInitializers(ExampleInitializer.class);
public class RootConnection implements ServiceConnection { ... }
public class ExampleService extends RootService {
    @Override
    public IBinder onBind(Intent intent) {
        // return IBinder from Messenger or AIDL stub implementation
    }
}
RootConnection connection = new RootConnection();
Intent intent = new Intent(context, ExampleService.class);
RootService.bind(intent, connection);
