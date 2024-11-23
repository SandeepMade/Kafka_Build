# Kafka_Build
 import com.jcraft.jsch.*;

import java.io.*;
import java.util.Properties;

public class PuttyServerAutomationWithScript {

    public static void main(String[] args) {
        String user = "your-username"; // Your server username
        String host = "your-host"; // Server IP or hostname
        int port = 22; // SSH port
        String privateKeyPath = "C:/path/to/id_rsa"; // Path to your private key
        String scriptFilePath = "C:/path/to/commands.spsl"; // Path to the .spsl file

        try {
            // Setup JSch
            JSch jsch = new JSch();
            jsch.addIdentity(privateKeyPath);

            // Create a session
            Session session = jsch.getSession(user, host, port);

            // Disable strict host key checking
            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            session.setConfig(config);

            // Connect to the server
            session.connect();
            System.out.println("Connected to the server!");

            // Read and execute commands from the .spsl file
            try (BufferedReader br = new BufferedReader(new FileReader(scriptFilePath))) {
                String command;
                while ((command = br.readLine()) != null) {
                    if (!command.trim().isEmpty()) { // Skip empty lines
                        System.out.println("Executing: " + command);
                        executeCommand(session, command);
                    }
                }
            }

            // Disconnect the session
            session.disconnect();
            System.out.println("Session disconnected.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void executeCommand(Session session, String command) {
        try {
            ChannelExec channel = (ChannelExec) session.openChannel("exec");
            channel.setCommand(command);
            channel.setErrStream(System.err);

            InputStream in = channel.getInputStream();
            channel.connect();

            // Read and print command output
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(buffer)) != -1) {
                System.out.print(new String(buffer, 0, bytesRead));
            }

            channel.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

