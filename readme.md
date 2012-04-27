<h1>Overview:</h1>
The Flex-RESTkit allows for OAuth authentication and REST callouts to the Force.com platform.

<P>
<B>Warning:</B> Due to issues with the Adobe Flash Player, the REST API is essentially unavailable when embedding Flex into Visualforce.  This is because it strips out the Auth header and also ignores some of the basic REST HTTP actions.  Everything works as expected under AIR.

<BR /><BR />	
Also, due to Adobe's lack of official support for Flex - this project will probably not be updated.
</P>	

<H1>OAuth: Basic Usage</H1>
The OAuthConnection class will use the public key, private key and callback URL to direct the user through a dynamically generated web browser and hold the appropriate tokens in local storage for authentication.  It can also use the refresh token to maintain a persisten authentication, which the RESTConnection class utilizes by default.

You’ll need to import the class:

<PRE>import com.force.oauth.OAuthConnection;</PRE>

From there, just use the constructor:

<PRE>var oauth:OAuthConnection = new OAuthConnection("{PUBLICKEY}","{PRIVATEKEY}","{CALLBACK}”);</PRE>

After that you can have the connection start the process of creating a web view for the user, and then requesting and storing the necessary tokens. To do this, give the login method the main display stage and an aysncronous responder as a callback:

<PRE>oauth.login(this.stage,new mx.rpc.AsyncResponder(loginHandler,com.force.utility.util.genericError));</PRE>


<H1>REST: Basic Usage</H1>
The RESTConnection class can utilize either a session ID and server URL from logged in credentials, or, more commonly, work in tandem with OAuthConnection.

To use the session ID and server URL:

<PRE>
private function login():void {
    var oauthConnection:OAuthConnection = new OAuthConnection("PUBLICKEY","PRIVATEKEY","CALLBACK");
    oauthConnection.login(this.stage,new mx.rpc.AsyncResponder(postLogin,com.force.utility.util.genericError));
    }
             
public function postLogin(restResult:Object, token:Object):void {
       if(restResult != null) {
            var lr:LoginRequest = new LoginRequest(OAuthConnection.SOAPLoginRequest());
            lr.callback = new com.salesforce.AsyncResponder( search, com.force.utility.util.genericError );
            this.conn.login(lr);
            }
        }
             
public function search(result:Object):void {
         //  continue implementation code as normal here
}
</PRE>

<H2>And to use OAuth:</H2>

<PRE>
private function login():void {
    rest = new RESTConnection();
    rest.oauthConnection = new OAuthConnection("{PUBLICKEY}","{PRIVATEKEY}","{CALLBACK}");
    rest.oauthConnection.login(this.stage,new mx.rpc.AsyncResponder(loginHandler,com.force.utility.util.genericError));
    }
             
private function loginHandler(loginResult:Object, token:Object):void {
    rest.oauth = loginResult;
    rest.query("SELECT ID, Name from Contact LIMIT 5",new mx.rpc.AsyncResponder(queryHandler,com.force.utility.util.genericError));
    }</PRE>

<H2>Oauth for mobile</H2> 
Uses a slightly different class, as the dynamically generated browser can't use HTMLLoader:
<PRE>
private function login():void {
        rest = new RESTConnection();
        rest.oauthConnection = new OAuthConnectionMobile(“PUBLICKEY","PRIVATEKEY","CALLBACKURL");
        rest.oauthConnection.login(this.stage,new mx.rpc.AsyncResponder(loginHandler,com.force.utility.util.genericError));
        }
</PRE>   

<H1>Quick REST Examples</H1>
<I>Query:</I>
<PRE>
rest.query("SELECT ID, Name from Contact LIMIT 5",new mx.rpc.AsyncResponder(queryHandler,com.force.utility.util.genericError));
			}
			
private function queryHandler(queryResult:Array, token:Object):void {
	trace("1::"+ObjectUtil.toString(queryResult));
	}
</PRE>
<I>Get Object By Id</I>
<PRE>
rest.getObjectById(“Account”,”{ACCOUNTID}”,new mx.rpc.Responder(handleResult,com.force.utility.util.genericError));

private function handleResult(result:Object,token:Object):void {
	trace(result.Name);
}
</PRE>
<I>Creating data</I>
<PRE>
var newContact:Object = new Object();
newContact.FirstName = "Jane";
newContact.LastName = "Jones";
rest.create(newContact,"Contact",new AsyncResponder(com.force.utility.util.genericTrace,com.force.utility.util.genericError));
</PRE>
<I>Updating data</I>
<PRE>oldContact.FirstName = "Jane";
oldContact.LastName = "Smith";
rest.create(oldContact,"Contact",new AsyncResponder(com.force.utility.util.genericTrace,com.force.utility.util.genericError));</PRE>
<I>Accessing a File</I>
<PRE>rest.getFileByURI({URI},new AsyncResponder(com.force.utility.util.genericTrace,com.force.utility.util.genericError));
</PRE>

<B>Deleting data</B>
<P>
Since Flex does not natively support the DELETE HTTP method, the library needs to use some third party tools to create a proper delete request for the REST API.  To do this, you need to install the following libraries:
<UL>
 <LI>as3httpclientlib @ <a href="http://code.google.com/p/as3httpclientlib/">code.google</a></LI>
 <LI>as3corelib @ <a href="https://github.com/mikechambers/as3corelib">github</a></LI>
 <LI>as3crypto @ <a href="http://code.google.com/p/as3crypto/">code.google</a></LI>
</UL>

The delete request also works differently to distinct this portion of the code the rest of the library that doesn't require the above libraries.  To perform a delete request, first create a DeleteRequest object and pass in your RESTConnection:

<PRE>var delReq:DeleteRequest = new DeleteRequest(rest);</PRE>

After that, the DeleteRequest behaves in a similar manner as the rest of the library:

<PRE>delReq.deleteObject(resultGrid.selectedItem.Id,"Contact",new AsyncResponder(refreshData,com.force.utility.util.genericError));</PRE>		

</P>
