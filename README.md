1\. Essential Podman Commands
-----------------------------

### Image Management

*   **Search**: podman search ubi9 (Search for Universal Base Images).  
    
*   **Pull**: podman pull .  
    
*   **Login**: podman login registry.redhat.io.  
    
*   **List**: podman images or podman image list.  
    
*   **Inspect**: podman image inspect .  
    
*   **Remove**: podman image rm .  
    
*   **Prune**: podman image prune (Removes unused images).  
    

### Container Lifecycle

*   **Run**: podman run -it  /bin/bash (Interactive mode).  
    
*   **Detached**: podman run -d .  
    
*   **Status**: podman ps (Running) or podman ps -a (All).  
    
*   **Stop/Start**: podman stop / podman start .  
    
*   **Execution**: podman exec -it sh.  
    
*   **Commit**: podman commit localhost/image:tag (Create image from a running container).  
    

> **Note:** A container stops automatically once its main application stops.
>
> 2\. Container Networking & Storage
----------------------------------

### Networking

*   **Rootless Limitation**: Rootless containers do not have their own IP address; they map to host ports.  
    
*   **Port Forwarding**: podman run -p 127.0.0.1:8080:80 nginx (Restricts access to localhost only).  
    
*   **Troubleshooting**: Use podman exec ss -pant to check internal listening ports.  
    

### Persistent Storage

*   **Volumes**: Independent objects managed by Podman.  
    
*   **Bind Mount**: podman run -v /host/path:/container/path:Z .  
    
*   **Volume Commands**:
    
    *   podman volume create .  
        
    *   podman volume export --output .
 
### Key Instructions

**InstructionPurposeFROM**

Defines the base image.

**WORKDIR**

Sets the working directory for subsequent commands.

**COPY**

Copies files from host to image.

**RUN**

Executes commands during the build process (creates a new layer).

**ENTRYPOINT**

The main executable to run.

**CMD**

Default arguments for the Entrypoint.

**USER**

Defines the UID/User that runs the process.


4\. Systemd Integration
-----------------------

To ensure containers start automatically on boot:

1.  **Generate Service**: podman generate systemd --name \--files --new.  
    
2.  **Move File**: Copy to ~/.config/systemd/user/ for rootless users.  
    
3.  **Enable Linger**: loginctl enable-linger (Allows user services to run without an active session).  
    
4.  **Enable Service**: systemctl --user enable --now container-.service.  
    

5\. Multi-Container Orchestration
---------------------------------

### Podman Compose

Used to define and run multi-container applications using compose.yaml.  

*   podman-compose up -d: Start in background.  
    
*   podman-compose down -v: Remove containers and associated volumes.  
    

### OpenShift (OC)

*   **New Project**: oc new-project .  
    
*   **Deployment**: oc create deploy \--image= --replicas=3.  
    
*   **Expose Service (Internal)**: oc expose deployment .  
    
*   **Expose Route (External)**: oc expose service .  
    

6\. Troubleshooting Summary
---------------------------

*   **Exit Code 0**: Success.  

Podman Compose & Kubernetes YAML (EX188)
----------------------------------------

### Podman Compose

*   Uses compose.yaml / compose.yml to define **one or more containers**
    
*   Suitable for **local development only**
    
*   ❌ Not recommended for production
    
*   ✅ Use **Kubernetes** / OpenShift in production
    

### Key Commands

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   podman-compose up   `

*   Creates and starts containers defined in the compose file
    

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   podman-compose up -d   `

*   Runs containers in **detached mode**
    

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   podman generate kube   `

*   Generates Kubernetes YAML from existing containers or pods
    

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`podman play kube` 

*   Creates containers from Kubernetes YAML files
    

### Compose File Structure

*   **services** – Container definitions
    
*   **volumes** – Persistent storage
    
*   **networks** – Custom container networks
    

### Common Service Properties

*   image
    
*   container\_name
    
*   ports (host:container)
    
*   networks
    
*   volumes
    
*   environment
    

### Minimal compose.yaml Example

Podman Compose & Kubernetes YAML (EX188)
----------------------------------------

### Podman Compose

*   Uses compose.yaml / compose.yml to define **one or more containers**
    
*   Suitable for **local development only**
    
*   ❌ Not recommended for production
    
*   ✅ Use **Kubernetes** / OpenShift in production
    


    
