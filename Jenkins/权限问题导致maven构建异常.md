```verilog
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project videobds-dispose: Execution default-compile of goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile failed: Plugin org.apache.maven.plugins:maven-compiler-plugin:3.1 or one of its dependencies could not be resolved: The following artifacts could not be resolved: org.codehaus.plexus:plexus-container-default:jar:1.5.5, org.apache.xbean:xbean-reflect:jar:3.4, commons-logging:commons-logging-api:jar:1.1, com.google.collections:google-collections:jar:1.0, junit:junit:jar:3.8.2: Could not transfer artifact org.codehaus.plexus:plexus-container-default:jar:1.5.5 from/to nexus (http://10.10.50.204:8081/nexus/content/groups/public/): /software/repos/org/codehaus/plexus/plexus-container-default/1.5.5/plexus-container-default-1.5.5.jar.part.lock (Permission denied) -> [Help 1]

```

原因为：**/software/repos/org/codehaus/plexus****目录的所有者是root****，而不是jenkins，删掉重新下载即可**

