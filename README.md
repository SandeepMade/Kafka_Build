# Kafka_Build
dependencies {
    // JSch library for SSH
    implementation 'com.jcraft:jsch:0.1.55'
    
    // JSch-AgentProxy to enable agent support
    implementation 'com.github.mwiede:jsch-agent-proxy:0.0.9'
    
    // Connector for Pageant on Windows
    implementation 'com.github.mwiede:jsch-agent-proxy-pageant:0.0.9'
    
    // Optional: JNR libraries for Unix-like OS, only needed for non-Windows setups
    implementation 'com.github.jnr:jnr-ffi:2.1.10'
    implementation 'com.github.jnr:jnr-unixsocket:0.38.18'
}
