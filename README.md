# Kafka_Build
import com.jcraft.jsch.*;

import java.io.*;
import java.util.Properties;

public class DynamicIsaacPodLogFetcher {

    public static void main(String[] args) {
        String user = "your-username"; // Your server username
        String host = "your-host"; // Server IP or hostname
        int port = 22; // SSH port
        String privateKeyPath = "C:/path/to/id_rsa"; // Path to your private key

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

            // Step 1: Execute the commands in sequence
            String[] commands = {
                "kubectl config get-contexts",
                "kubectl config use-context tstl-smp",
                "kubectl get ns",
                "kubectl config set-context --current --namespace=com-att-wfe-smp",
                "kubectl get pods --no-headers" // Get pods list
            };

            String podListOutput = executeCommandsAndCaptureOutput(session, commands);

            // Step 2: Find the dynamic 'isaac' pod name
            String isaacPodName = findIsaacPodName(podListOutput);
            if (isaacPodName == null) {
                System.out.println("No pod found starting with 'isaac'.");
            } else {
                System.out.println("Found pod: " + isaacPodName);

                // Step 3: Fetch logs for the 'isaac' pod
                String logCommand = "kubectl logs " + isaacPodName;
                String logs = executeCommandAndCaptureOutput(session, logCommand);
                System.out.println("Logs for pod " + isaacPodName + ":\n" + logs);
            }

            // Disconnect the session
            session.disconnect();
            System.out.println("Session disconnected.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String executeCommandsAndCaptureOutput(Session session, String[] commands) {
        StringBuilder output = new StringBuilder();
        try {
            for (String command : commands) {
                System.out.println("Executing: " + command);
                output.append(executeCommandAndCaptureOutput(session, command));
                output.append("\n");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return output.toString();
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

