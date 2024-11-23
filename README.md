# Kafka_Build
 import com.jcraft.jsch.*;

import java.io.*;
import java.util.Properties;

public class DynamicPodConnector {

    public static void main(String[] args) {
        String user = "your-username"; // Your server username
        String host = "your-host"; // Server IP or hostname
        int port = 22; // SSH port
        String privateKeyPath = "C:/path/to/id_rsa"; // Path to your private key
        String spplFilePath = "C:/path/to/commands.sppl"; // Path to the SPPL file containing commands

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

            // Step 1: Read commands from SPPL file
            String podListCommand = readCommandFromSPPL(spplFilePath, "get_pods");
            String podListOutput = executeCommandAndCaptureOutput(session, podListCommand);

            // Step 2: Find pod starting with "isaac"
            String isaacPodName = findIsaacPodName(podListOutput);
            if (isaacPodName == null) {
                System.out.println("No pod found starting with 'isaac'.");
            } else {
                System.out.println("Found pod: " + isaacPodName);

                // Step 3: Execute commands for the identified pod
                String connectCommand = readCommandFromSPPL(spplFilePath, "connect_pod").replace("{podName}", isaacPodName);
                executeCommand(session, connectCommand);
            }

            // Disconnect the session
            session.disconnect();
            System.out.println("Session disconnected.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String readCommandFromSPPL(String filePath, String commandKey) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (line.startsWith(commandKey + "=")) {
                    return line.split("=", 2)[1].trim();
                }
            }
        }
        throw new IllegalArgumentException("Command with key '" + commandKey + "' not found in SPPL file.");
    }

    public static String executeCommandAndCaptureOutput(Session session, String command) {
        StringBuilder output = new StringBuilder();
        try {
            ChannelExec channel = (ChannelExec) session.openChannel("exec");
            channel.setCommand(command);
            channel.setErrStream(System.err);

            InputStream in = channel.getInputStream();
            channel.connect();

            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(buffer)) != -1) {
                output.append(new String(buffer, 0, bytesRead));
            }

            channel.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return output.toString();
    }

    public static void executeCommand(Session session, String command) {
        try {
            ChannelExec channel = (ChannelExec) session.openChannel("exec");
            channel.setCommand(command);
            channel.setErrStream(System.err);

            InputStream in = channel.getInputStream();
            channel.connect();

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

    public static String findIsaacPodName(String podListOutput) {
        try (BufferedReader reader = new BufferedReader(new StringReader(podListOutput))) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (line.startsWith("isaac")) {
                    return line.split("\\s+")[0]; // Return the first column (pod name)
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null; // No pod found
    }
}
