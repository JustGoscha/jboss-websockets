WebSockets for JBoss AS 7.1.2+
------------------------------

NOTE: This does not currently work on versions of AS older than 7.1.2. It also requires use of the Apache Portable
Runtime connector (APR). This limitation will be addressed in a future version of AS.

To Configure APR in JBoss AS 7.1.x:

 1. In domain/configuration/domain.xml ->
     Change line :: <subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
              to :: <subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="true">

 2. If you are on OS X you may need to tweak JAVA_OPTS a bit:
   :: export JAVA_OPTS="-d32 -Xmx512m -Djboss.modules.system.pkgs=sun.nio.ch -Djava.library.path=$JBOSS_HOME/bin/native"
       - The native APR connector binaries for OS X are only 32-bit
       - The Mac JRE NIO libraries try to access sun.nio.ch.* classes, and JBoss Modules is angry about that.
       - Add the bin/native dir to the JVM library search path. This assumes you've set $JBOSS_HOME. If not, just fully qualify path.




Using the WebsocketServlet:
---------------------------

Once AS 7.1.2+ is configured as specified above, using websockets is as simple as implementing a Servlet which
extends org.jboss.as.websockets.servlet.WebsocketServlet and implements its abstract methods. You map the servlet
as you normally would, and that servlet mapping will become the upgradable WebSocket path.

Example Implementation:

    @WebServlet("/websocket/")
    public class MyWebSocketServlet extends WebsocketServlet {

      @Override
      protected void onSocketOpened(HttpEvent event, WebSocket socket) throws IOException {
        System.out.println("Websocket opened :)");
      }

      @Override
      protected void onSocketClosed(HttpEvent event) throws IOException {
        System.out.println("Websocket closed :(");
      }

      @Override
      protected void onReceivedTextFrame(HttpEvent event, final WebSocket socket, String text) throws IOException {
        if ("Hello".equals(text)) {
          socket.writeTextFrame("Hey, there!");
        }
      }
    }


And, that's about it except for one point of errata: the session information is not automatically shared between
the HTTP and the WebSocket session. So when an upgrade occurs, a new HttpSession will be opened. You will need to
write your own code to associate the new session with the old session.

Note that binary frames are not supported yet. Which is okay since no browsers actually support them yet. =)