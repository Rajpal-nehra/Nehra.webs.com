import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;
import java.io.*;
import java.net.InetSocketAddress;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
public class SimpleServer {
    public static void main(String[] args) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress("0.0.0.0", 5000), 0);
        server.createContext("/", new StaticFileHandler());
        server.setExecutor(null);
        System.out.println("Server started on http://0.0.0.0:5000");
        server.start();
    }
    static class StaticFileHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            String requestPath = exchange.getRequestURI().getPath();
            
            // Default to index.html for root path
            if ("/".equals(requestPath)) {
                requestPath = "/index.html";
            }
            
            // Remove leading slash
            String filePath = requestPath.substring(1);
            Path path = Paths.get(filePath);
            
            if (Files.exists(path) && !Files.isDirectory(path)) {
                String contentType = getContentType(filePath);
                exchange.getResponseHeaders().set("Content-Type", contentType);
                
                byte[] fileBytes = Files.readAllBytes(path);
                exchange.sendResponseHeaders(200, fileBytes.length);
                
                OutputStream os = exchange.getResponseBody();
                os.write(fileBytes);
                os.close();
            } else {
                String notFound = "404 - File not found";
                exchange.sendResponseHeaders(404, notFound.length());
                OutputStream os = exchange.getResponseBody();
                os.write(notFound.getBytes());
                os.close();
            }
        }
        
        private String getContentType(String fileName) {
            if (fileName.endsWith(".html")) return "text/html";
            if (fileName.endsWith(".css")) return "text/css";
            if (fileName.endsWith(".js")) return "application/javascript";
            if (fileName.endsWith(".png")) return "image/png";
            if (fileName.endsWith(".jpg") || fileName.endsWith(".jpeg")) return "image/jpeg";
            if (fileName.endsWith(".mp4")) return "video/mp4";
            return "text/plain";
        }
    }
}
