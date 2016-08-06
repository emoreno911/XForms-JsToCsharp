# XForms-JsToCsharp
How to communicate between JavaScript and WebView Native Code in Xamarin Forms

##### 1 - Create a new Webview Control class which inherited from Webview
```csharp
 public class MyWebView : WebView
 {
   Action<string, string> action;
  
  
   public void RegisterAction (Action<string, string> callback)
   {
     action = callback;
   }
  
   public void Cleanup ()
   {
     action = null;
   }
  
   public void InvokeAction (string actionName, string data)
   {
     if (action == null || actionName == null) {
       return;
     }
     action.Invoke (actionName, data);
   }
  
   public void ExecJavaScriptFunction(string jsCode)
   {
     Device.BeginInvokeOnMainThread (() => this.Eval(jsCode));
   }
 }
```

##### 2 - Use the Webview control in your page
```csharp
_webView = new MyWebView()
   {
     VerticalOptions = LayoutOptions.FillAndExpand,
   };

Content = _webView;
_webView.RegisterAction (ProcessWebMessage); //register the action handler
```

##### 3 - The ProcessWebMessage function
```csharp
private void ProcessWebMessage(string actionName, string data)
{

  if(actionName.ToLower() == "someaction")
  {
    //do you actions here
    _webView.ExecJavaScriptFunction ("callBackJs('message...')");
  }

}
```

##### 4 - Andriod Custom Renderer
```csharp
public class MyWebViewRenderer : WebViewRenderer
{
  protected override void OnElementChanged(ElementChangedEventArgs<Xamarin.Forms.WebView> e)
  {
    base.OnElementChanged(e);

    Android.Webkit.WebView webView = this.Control;
    webView.SetWebChromeClient(new Android.Webkit.WebChromeClient());
    webView.Settings.UseWideViewPort = true;
    webView.Settings.JavaScriptEnabled = true;


    if(e.OldElement != null)
    {
      Control.RemoveJavascriptInterface ("jsBridge");
      var hybridWebView = e.OldElement as MyWebView;
      hybridWebView.Cleanup ();
    }

    if (e.NewElement != null) {
      Control.AddJavascriptInterface (new JSBridge (this), "jsBridge");
    }
  }

  public void InvokeAction(string aName, string data)
  {
    var hybridWebView = this.Element as MyWebView;
    hybridWebView.InvokeAction (aName, data);
  }

}

public class JSBridge : Java.Lang.Object
{
  readonly WeakReference<MyWebViewRenderer> hybridWebViewRenderer;

  public JSBridge (MyWebViewRenderer hybridRenderer)
  {
    hybridWebViewRenderer = new WeakReference <MyWebViewRenderer> (hybridRenderer);
  }

  [JavascriptInterface]
  [Export ("invokeAction")]
  public void InvokeAction (string aName, string data)
  {
    MyWebViewRenderer hybridRenderer;

    if (hybridWebViewRenderer != null && hybridWebViewRenderer.TryGetTarget (out hybridRenderer)) {
      hybridRenderer.InvokeAction (aName, data);
    }
  }
}
```

##### 5 - iOS custom renderer
```csharp
public class MyWebViewRenderer : WebViewRenderer
{
  protected override void OnElementChanged(VisualElementChangedEventArgs e)
  {

    base.OnElementChanged(e);

    if (e.OldElement == null)
    {
      var _MyWebView = (MyWebView)this.Element;
      Delegate = new MyWebViewDelegate(_MyWebView);
    }
  }		
}

internal class MyWebViewDelegate : UIWebViewDelegate
{
  private MyWebView _MyWebView;

  public MyWebViewDelegate(MyWebView MyWebView)
  {
    _MyWebView = MyWebView;
  }


  public override bool 
    ShouldStartLoad(UIWebView webView, NSUrlRequest request, UIWebViewNavigationType navigationType)
  {
    string urlString = request.Url.AbsoluteString;

    if (!string.IsNullOrEmpty (urlString)) {
      string clientJsProc = "clientjscall://";
      if (urlString.StartsWith (clientJsProc, StringComparison.InvariantCultureIgnoreCase)) {
        var actionParams = urlString.Substring (clientJsProc.Length);
        string[] temp = actionParams.Split (new string[]{ "/" }, StringSplitOptions.RemoveEmptyEntries);
        if (temp.Length == 2) {
          _MyWebView.InvokeAction (temp [0], temp [1]);
        }
        return false;
      }
    }

    return true;
  }

}
```

##### 6 - in your html page
```javascript
function invokeCSCode(actionName, data) {
	try {
	  //  Android
		jsBridge.invokeAction(actionName, data);
	}
	catch (err) {
		//iOS
		execIOSJs(actionName, data);
	}
}

function execIOSJs(actionName, data) {
	try {
		var iframe = document.createElement("iframe");
		iframe.setAttribute("src", "clientjscall://" + actionName + "/" + data);
		iframe.setAttribute("height", "1px");
		iframe.setAttribute("width", "1px");
		document.documentElement.appendChild(iframe);
		iframe.parentNode.removeChild(iframe);
		iframe = null;
	}
	catch (err) {
		alert(err);
	}
}

function callBackJs(msg) {
	
}
```

Taken from [here](http://www.tipsabc.com/2016/xamarin-forms-communicate-javascript-webview-native-code/) to keep archived.
