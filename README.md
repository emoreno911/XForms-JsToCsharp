# XForms-JsToCsharp (Android)
How to communicate between JavaScript and WebView Native Code in Xamarin Forms

##### 1 - Create a Webview Custom Renderer for your Andriod project
```csharp
public class JStoCsharpWebviewRenderer : WebViewRenderer
{
    protected override void OnElementPropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        if (Control != null)
        {
            Control.Settings.JavaScriptEnabled = true;
            Control.AddJavascriptInterface(new JSBridge(Forms.Context), "CSharp");
        }
        base.OnElementPropertyChanged(sender, e);
    }

}
```

##### 2 - The next class will be a bridge between Javascript and CSharp
```csharp
public class JSBridge : Java.Lang.Object
{
    Context context;

    public JSBridge(Context context)
    {
        this.context = context;
    }

    [Export]
    [JavascriptInterface]
    public void ShowToast(string msg)
    {
        Toast.MakeText(context, msg, ToastLength.Short).Show();
    }

}
```

##### 3 - Create a new webview control in your Xamarin.Forms project
```csharp
public class JStoCsharpWebviewRenderer : WebView
{
    public JStoCsharpWebviewRenderer()
    {
        const string html = @"
        <html>
          <body>
            <h3>Demo calling C# from JavaScript</h3>
            <button type=""button"" onClick=""CSharp.ShowToast('Hello from JS')"">Native Interaction</button>
          </body>
        </html>";

        var htmlSource = new HtmlWebViewSource();
        htmlSource.BaseUrl = DependencyService.Get<IBaseUrl>().Get();

        htmlSource.Html = html;
        Source = htmlSource;
    }
}
```

##### 4 - Then test
```csharp
public App()
{
    MainPage = new ContentPage { 
        Content = new JStoCsharpWebviewRenderer() 
    };
}
```
