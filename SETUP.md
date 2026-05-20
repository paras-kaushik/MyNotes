You’re on a **new Apple Silicon MacBook Pro M5 Pro**, so we’ll set it up cleanly for:

- **Java DSA practice**
- **Spring Boot backend**
- **PostgreSQL**
- **React / Angular**
- **AWS basics**
- **Git + IDE tooling**

Use **ARM64 / Apple Silicon** versions wherever possible. Avoid random Intel/x86 installers unless absolutely needed.

---

# 0. Setup map

```text
MacBook M5 Pro
   |
   ├── Xcode Command Line Tools
   ├── Homebrew
   ├── Git
   ├── SDKMAN
   │      ├── Java 21 default
   │      ├── Java 17 optional
   │      ├── Maven
   │      └── Gradle
   ├── IntelliJ IDEA
   ├── VS Code
   ├── Docker Desktop
   │      └── PostgreSQL container
   ├── Node via nvm
   │      ├── React practice
   │      └── Angular practice
   └── AWS CLI
```

For Apple Silicon Macs, Homebrew’s supported default prefix is `/opt/homebrew`, and tools like Docker Desktop, VS Code, and IntelliJ have Apple Silicon / Arm64 support. ([docs.brew.sh](https://docs.brew.sh/Installation.html?utm_source=openai))

---

# 1. Open Terminal and verify your Mac architecture

Open **Terminal** and run:

```bash
uname -m
```

Expected:

```text
arm64
```

If you see `arm64`, you are on Apple Silicon, which is correct for M-series Macs.

Also check your shell:

```bash
echo $SHELL
```

Expected:

```text
/bin/zsh
```

Modern macOS uses `zsh` by default.

---

# 2. Install Xcode Command Line Tools

These provide developer basics like `git`, compilers, headers, and build tools. Apple supports installing Command Line Tools from Terminal using `xcode-select`. ([developer.apple.com](https://developer.apple.com/documentation/xcode/installing-the-command-line-tools/?utm_source=openai))

Run:

```bash
xcode-select --install
```

A popup will appear. Click **Install**.

After installation, verify:

```bash
xcode-select -p
git --version
```

Expected path:

```text
/Library/Developer/CommandLineTools
```

If it says tools are already installed, that’s fine.

---

# 3. Install Homebrew

Homebrew is the package manager we’ll use for apps and CLI tools. On Apple Silicon, Homebrew installs under `/opt/homebrew`. ([docs.brew.sh](https://docs.brew.sh/Installation.html?utm_source=openai))

Run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

When installation finishes, run these:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:

```bash
brew --version
brew doctor
```

If `brew doctor` says:

```text
Your system is ready to brew.
```

you’re good.

---

# 4. Install essential CLI utilities

Run:

```bash
brew install git wget curl tree jq ripgrep htop
```

What these are for:

| Tool | Why useful |
|---|---|
| `git` | Version control |
| `wget`, `curl` | Download / test APIs |
| `tree` | Visualize folder structures |
| `jq` | Pretty-print JSON |
| `ripgrep` | Fast code search |
| `htop` | Process / CPU monitoring |

Verify:

```bash
git --version
tree --version
jq --version
```

---

# 5. Configure Git

Run:

```bash
git config --global user.name "Paras Kaushik"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
```

Check config:

```bash
git config --global --list
```

Optional but recommended: generate SSH key for GitHub.

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Press Enter for defaults.

Copy public key:

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

Then add it to GitHub under **Settings → SSH and GPG Keys**.

---

# 6. Install Java correctly using SDKMAN

For interview preparation, do **not** install Java randomly from multiple sources. Use **SDKMAN** because it lets you switch between Java versions cleanly. SDKMAN supports installing and switching Java versions on macOS, and Eclipse Temurin builds are available for macOS AArch64 / Apple Silicon. ([sdkman.io](https://sdkman.io/install/?utm_source=openai))

Install SDKMAN:

```bash
curl -s "https://get.sdkman.io" | bash
```

Then initialize it:

```bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Verify:

```bash
sdk version
```

---

## 6.1 Which Java version should you install?

For your Gartner role, use:

```text
Java 21 = default
Java 17 = optional fallback
```

Why?

- Java 17, 21, and 25 are LTS releases.
- Many enterprise Spring Boot systems still use Java 17 or 21.
- Java 21 is modern, stable, and ideal for interview practice.
- Spring Boot 3 requires Java 17+, and Spring Boot 3.0.x is documented as compatible up to Java 21. ([oracle.com](https://www.oracle.com/europe/java/technologies/java-se-support-roadmap.html?utm_source=openai))

Install Java 21:

```bash
sdk install java 21-tem
```

Install Java 17 too:

```bash
sdk install java 17-tem
```

Set Java 21 as default:

```bash
sdk default java 21-tem
```

If `21-tem` or `17-tem` does not work because SDKMAN version identifiers changed, run:

```bash
sdk list java
```

Then look for the latest Temurin Java 21 identifier, for example:

```text
21.0.x-tem
```

Then install using that exact identifier:

```bash
sdk install java 21.0.x-tem
sdk default java 21.0.x-tem
```

---

## 6.2 Set `JAVA_HOME`

Add this to your shell config:

```bash
echo 'export JAVA_HOME="$HOME/.sdkman/candidates/java/current"' >> ~/.zshrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Verify Java:

```bash
which java
java -version
echo $JAVA_HOME
```

Expected `which java` should look like:

```text
/Users/paras/.sdkman/candidates/java/current/bin/java
```

Expected Java version should show Java 21.

---

# 7. Install Maven and Gradle

Maven and Gradle are the two main Java build tools. Maven officially supports installation on macOS via Homebrew or SDKMAN, and Gradle also documents installation via SDKMAN or Homebrew. ([maven.apache.org](https://maven.apache.org/install?utm_source=openai))

Use SDKMAN:

```bash
sdk install maven
sdk install gradle
```

Verify:

```bash
mvn -v
gradle -v
```

You should see Java 21 being used.

Important: Many real projects use Maven/Gradle wrappers:

```text
./mvnw
./gradlew
```

For interview practice, knowing Maven is more important than deep Gradle.

---

# 8. Install IDEs

## Recommended setup

```text
IntelliJ IDEA = primary Java/Spring Boot IDE
VS Code       = frontend + quick scripts + DSA notes
```

Install IntelliJ IDEA Ultimate if you can use the trial. It has better Spring Boot support. If not, Community Edition is okay for Java/Maven/Gradle practice.

### Option A: IntelliJ IDEA Ultimate

```bash
brew install --cask intellij-idea
```

### Option B: IntelliJ IDEA Community

```bash
brew install --cask intellij-idea-ce
```

Install VS Code:

```bash
brew install --cask visual-studio-code
```

VS Code supports macOS Arm64 builds for Apple Silicon. ([code.visualstudio.com](https://code.visualstudio.com/docs/setup/mac?source=post_page---------------------------&utm_source=openai))

---

## 8.1 Enable `code` command for VS Code

Open VS Code.

Press:

```text
Cmd + Shift + P
```

Search:

```text
Shell Command: Install 'code' command in PATH
```

Then verify:

```bash
code --version
```

---

## 8.2 Useful VS Code extensions

Run:

```bash
code --install-extension vscjava.vscode-java-pack
code --install-extension vmware.vscode-spring-boot
code --install-extension vscjava.vscode-spring-boot-dashboard
code --install-extension redhat.java
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint
```

---

# 9. Install Docker Desktop

Use Docker for PostgreSQL instead of installing PostgreSQL directly on your Mac. This avoids messy local DB setup.

Docker Desktop provides a Mac Apple Silicon download. ([docs.docker.com](https://docs.docker.com/get-started/introduction/get-docker-desktop/?utm_source=openai))

Install:

```bash
brew install --cask docker
```

Open Docker:

```bash
open /Applications/Docker.app
```

Wait until Docker says it is running.

Verify:

```bash
docker --version
docker compose version
```

If Docker asks for permissions, approve them.

---

# 10. Start PostgreSQL using Docker

Create a workspace:

```bash
mkdir -p ~/interview-prep/postgres
cd ~/interview-prep/postgres
```

Create Docker Compose file:

```bash
cat > docker-compose.yml <<'YAML'
services:
  postgres:
    image: postgres:16
    container_name: pg-interview
    environment:
      POSTGRES_DB: interviewdb
      POSTGRES_USER: interview
      POSTGRES_PASSWORD: interview
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
YAML
```

Start PostgreSQL:

```bash
docker compose up -d
```

Check running container:

```bash
docker ps
```

Connect to PostgreSQL:

```bash
docker exec -it pg-interview psql -U interview -d interviewdb
```

Inside `psql`, test:

```sql
select version();
```

Exit:

```sql
\q
```

---

## 10.1 Useful PostgreSQL commands

Start DB:

```bash
docker compose up -d
```

Stop DB:

```bash
docker compose down
```

Stop and delete data:

```bash
docker compose down -v
```

Connect:

```bash
docker exec -it pg-interview psql -U interview -d interviewdb
```

Run SQL directly:

```bash
docker exec -it pg-interview psql -U interview -d interviewdb -c "select current_database();"
```

---

# 11. Install DBeaver and Postman

DBeaver helps you inspect PostgreSQL visually.

Postman helps test REST APIs.

Install:

```bash
brew install --cask dbeaver-community
brew install --cask postman
```

DBeaver PostgreSQL connection:

```text
Host: localhost
Port: 5432
Database: interviewdb
Username: interview
Password: interview
```

---

# 12. Create Java DSA practice folder

Create a simple Java practice workspace:

```bash
mkdir -p ~/interview-prep/dsa-java
cd ~/interview-prep/dsa-java
```

Create first Java file:

```bash
cat > Main.java <<'JAVA'
import java.util.*;

public class Main {

    public static void main(String[] args) {
        int[] nums = {1, 2, 3, 4};

        int[] result = productExceptSelf(nums);

        System.out.println(Arrays.toString(result));
    }

    public static int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] result = new int[n];

        int leftProduct = 1;

        for (int i = 0; i < n; i++) {
            result[i] = leftProduct;
            leftProduct = leftProduct * nums[i];
        }

        int rightProduct = 1;

        for (int i = n - 1; i >= 0; i--) {
            result[i] = result[i] * rightProduct;
            rightProduct = rightProduct * nums[i];
        }

        return result;
    }
}
JAVA
```

Compile:

```bash
javac Main.java
```

Run:

```bash
java Main
```

Expected:

```text
[24, 12, 8, 6]
```

This confirms Java is working.

---

# 13. Recommended DSA Java template

Use this for quick practice:

```java
import java.util.*;

public class Main {

    public static void main(String[] args) {
        // Test input here
    }

    /*
     * Pattern reminders:
     *
     * HashMap:
     * Map<Integer, Integer> map = new HashMap<>();
     *
     * HashSet:
     * Set<Character> set = new HashSet<>();
     *
     * Stack:
     * Deque<Character> stack = new ArrayDeque<>();
     *
     * Queue:
     * Queue<int[]> queue = new ArrayDeque<>();
     *
     * Min heap:
     * PriorityQueue<Integer> minHeap = new PriorityQueue<>();
     *
     * Max heap:
     * PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
     */
}
```

---

# 14. Create a Spring Boot practice app

Now we’ll create a real Spring Boot backend connected to PostgreSQL.

Go to your prep folder:

```bash
mkdir -p ~/interview-prep
cd ~/interview-prep
```

Generate Spring Boot project:

```bash
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d javaVersion=21 \
  -d groupId=com.paras \
  -d artifactId=backend-practice \
  -d name=backend-practice \
  -d dependencies=web,validation,data-jpa,postgresql,actuator \
  -o backend-practice.zip
```

Unzip:

```bash
unzip backend-practice.zip -d backend-practice
cd backend-practice
```

Run once:

```bash
./mvnw spring-boot:run
```

It may fail because DB config is not added yet. Stop it with:

```text
Ctrl + C
```

---

## 14.1 Add PostgreSQL config

Create config file:

```bash
mkdir -p src/main/resources
cat > src/main/resources/application.yml <<'YAML'
spring:
  application:
    name: backend-practice

  datasource:
    url: jdbc:postgresql://localhost:5432/interviewdb
    username: interview
    password: interview

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
YAML
```

Make sure PostgreSQL container is running:

```bash
cd ~/interview-prep/postgres
docker compose up -d
```

Return to Spring app:

```bash
cd ~/interview-prep/backend-practice
```

---

# 15. Add a simple REST API

Create package folders:

```bash
mkdir -p src/main/java/com/paras/backendpractice/controller
mkdir -p src/main/java/com/paras/backendpractice/entity
mkdir -p src/main/java/com/paras/backendpractice/repository
mkdir -p src/main/java/com/paras/backendpractice/dto
```

---

## 15.1 Entity

```bash
cat > src/main/java/com/paras/backendpractice/entity/User.java <<'JAVA'
package com.paras.backendpractice.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    protected User() {
        // Required by JPA
    }

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
JAVA
```

---

## 15.2 Repository

```bash
cat > src/main/java/com/paras/backendpractice/repository/UserRepository.java <<'JAVA'
package com.paras.backendpractice.repository;

import com.paras.backendpractice.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {

    boolean existsByEmail(String email);
}
JAVA
```

---

## 15.3 Request DTO

```bash
cat > src/main/java/com/paras/backendpractice/dto/CreateUserRequest.java <<'JAVA'
package com.paras.backendpractice.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

public record CreateUserRequest(

        @NotBlank(message = "Name is required")
        String name,

        @Email(message = "Email must be valid")
        @NotBlank(message = "Email is required")
        String email
) {
}
JAVA
```

---

## 15.4 Response DTO

```bash
cat > src/main/java/com/paras/backendpractice/dto/UserResponse.java <<'JAVA'
package com.paras.backendpractice.dto;

import com.paras.backendpractice.entity.User;

public record UserResponse(
        Long id,
        String name,
        String email
) {
    public static UserResponse from(User user) {
        return new UserResponse(
                user.getId(),
                user.getName(),
                user.getEmail()
        );
    }
}
JAVA
```

---

## 15.5 Controller

```bash
cat > src/main/java/com/paras/backendpractice/controller/UserController.java <<'JAVA'
package com.paras.backendpractice.controller;

import com.paras.backendpractice.dto.CreateUserRequest;
import com.paras.backendpractice.dto.UserResponse;
import com.paras.backendpractice.entity.User;
import com.paras.backendpractice.repository.UserRepository;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {

        if (userRepository.existsByEmail(request.email())) {
            throw new IllegalArgumentException("Email already exists");
        }

        User user = new User(request.name(), request.email());

        User savedUser = userRepository.save(user);

        return UserResponse.from(savedUser);
    }

    @GetMapping
    public List<UserResponse> getAllUsers() {
        return userRepository.findAll()
                .stream()
                .map(UserResponse::from)
                .toList();
    }

    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("User not found"));

        return UserResponse.from(user);
    }
}
JAVA
```

---

## 15.6 Add health-check controller

```bash
cat > src/main/java/com/paras/backendpractice/controller/PingController.java <<'JAVA'
package com.paras.backendpractice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.Instant;
import java.util.Map;

@RestController
public class PingController {

    @GetMapping("/api/ping")
    public Map<String, Object> ping() {
        return Map.of(
                "message", "Spring Boot is running",
                "timestamp", Instant.now().toString()
        );
    }
}
JAVA
```

---

# 16. Run Spring Boot app

Make sure PostgreSQL is running:

```bash
cd ~/interview-prep/postgres
docker compose up -d
```

Run Spring app:

```bash
cd ~/interview-prep/backend-practice
./mvnw spring-boot:run
```

Test health:

```bash
curl http://localhost:8080/api/ping
```

Expected:

```json
{
  "message": "Spring Boot is running",
  "timestamp": "..."
}
```

Test actuator:

```bash
curl http://localhost:8080/actuator/health
```

Expected:

```json
{
  "status": "UP"
}
```

Create user:

```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Paras","email":"paras@example.com"}'
```

Get users:

```bash
curl http://localhost:8080/api/users
```

Check database:

```bash
docker exec -it pg-interview psql -U interview -d interviewdb -c "select * from users;"
```

If this works, your Java + Spring Boot + PostgreSQL environment is ready.

---

# 17. Install Node.js using nvm

For React/Angular practice, use `nvm`, not a random Node installer. `nvm` is designed to manage multiple Node versions on macOS. ([github.com](https://github.com/nvm-sh/nvm?utm_source=openai))

As of now, Node.js 24 is Active LTS and Node.js 22 is Maintenance LTS; production apps should use Active or Maintenance LTS versions. ([github.com](https://github.com/nodejs/Release?utm_source=openai))

Install nvm:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Reload shell:

```bash
source ~/.zshrc
```

If that does not work, close Terminal and reopen it.

Verify:

```bash
nvm --version
```

Install latest LTS Node:

```bash
nvm install --lts
nvm alias default 'lts/*'
```

Verify:

```bash
node -v
npm -v
```

Optional: install Node 22 too if you want compatibility with older projects:

```bash
nvm install 22
```

Switch versions:

```bash
nvm use 22
nvm use --lts
```

---

# 18. Create React practice app

Use Vite:

```bash
cd ~/interview-prep
npm create vite@latest react-practice -- --template react
cd react-practice
npm install
npm run dev
```

You’ll see something like:

```text
Local: http://localhost:5173/
```

Open it in browser.

---

# 19. Create Angular practice app

Install Angular CLI:

```bash
npm install -g @angular/cli
```

Verify:

```bash
ng version
```

Create app:

```bash
cd ~/interview-prep
ng new angular-practice --routing --style=scss
cd angular-practice
ng serve
```

Open:

```text
http://localhost:4200
```

---

# 20. Install AWS CLI

AWS CLI is useful for understanding S3, IAM, CloudWatch, ECS, Lambda, etc. AWS provides official macOS installation instructions for AWS CLI v2. ([docs.aws.amazon.com](https://docs.aws.amazon.com/en_us/cli/latest/userguide/getting-started-install.html?utm_source=openai))

Install using Homebrew:

```bash
brew install awscli
```

Verify:

```bash
aws --version
```

If you have an AWS account:

```bash
aws configure
```

Or for SSO-based company accounts:

```bash
aws configure sso
```

If you do not have AWS access, that is okay for interview prep. You mainly need conceptual understanding.

---

# 21. Install useful API / backend tools

```bash
brew install httpie
brew install watch
```

Test with HTTPie:

```bash
http GET http://localhost:8080/api/ping
```

Install browsers:

```bash
brew install --cask google-chrome
```

Optional:

```bash
brew install --cask insomnia
```

---

# 22. Add useful shell aliases

Run:

```bash
cat >> ~/.zshrc <<'EOF'

# Java
export JAVA_HOME="$HOME/.sdkman/candidates/java/current"
export PATH="$JAVA_HOME/bin:$PATH"

# Useful aliases
alias ll='ls -lah'
alias c='clear'
alias gs='git status'
alias ga='git add .'
alias gc='git commit -m'
alias gp='git push'
alias dps='docker ps'
alias dcup='docker compose up -d'
alias dcdown='docker compose down'

EOF

source ~/.zshrc
```

---

# 23. Final verification checklist

Run this full block:

```bash
echo "Architecture:"
uname -m

echo "\nHomebrew:"
brew --version

echo "\nGit:"
git --version

echo "\nJava:"
which java
java -version
echo "JAVA_HOME=$JAVA_HOME"

echo "\nMaven:"
mvn -v

echo "\nGradle:"
gradle -v

echo "\nNode:"
node -v
npm -v

echo "\nDocker:"
docker --version
docker compose version

echo "\nAWS:"
aws --version
```

Expected:

```text
Architecture: arm64
Java: 21.x
Maven: installed
Gradle: installed
Node: LTS
Docker: installed
AWS CLI: installed
```

---

# 24. Troubleshooting

## Problem: `brew: command not found`

Run:

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Then permanently fix:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
source ~/.zprofile
```

---

## Problem: wrong Java version

Check all Java binaries:

```bash
which -a java
java -version
sdk current java
```

Set Java 21 again:

```bash
sdk default java 21-tem
source ~/.zshrc
```

If `21-tem` fails:

```bash
sdk list java
```

Install the exact latest `21.x-tem`.

---

## Problem: IntelliJ does not detect Java

In IntelliJ:

```text
File → Project Structure → SDKs → + → JDK
```

Choose:

```text
/Users/paras/.sdkman/candidates/java/current
```

---

## Problem: Docker not running

Open Docker manually:

```bash
open /Applications/Docker.app
```

Wait until Docker Desktop says running.

Then:

```bash
docker ps
```

---

## Problem: PostgreSQL port already used

Check:

```bash
lsof -i :5432
```

If something else is using 5432, change your Docker Compose port:

```yaml
ports:
  - "5433:5432"
```

Then your JDBC URL becomes:

```yaml
url: jdbc:postgresql://localhost:5433/interviewdb
```

---

## Problem: `code` command not found

Open VS Code.

Press:

```text
Cmd + Shift + P
```

Run:

```text
Shell Command: Install 'code' command in PATH
```

Then restart Terminal.

---

# 25. Minimal setup if you are short on time

If you only have limited time before interview, prioritize:

```bash
# 1. Xcode tools
xcode-select --install

# 2. Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 3. SDKMAN + Java
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 21-tem
sdk install maven

# 4. IntelliJ + Docker + Postman
brew install --cask intellij-idea docker postman

# 5. Node LTS
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.zshrc
nvm install --lts
```

Then practice:

```text
DSA in Java
Spring Boot REST APIs
PostgreSQL queries
React/Angular basics
```

---

# 26. Your next action

Do this now in order:

```text
1. Install Xcode Command Line Tools
2. Install Homebrew
3. Install SDKMAN
4. Install Java 21
5. Install IntelliJ
6. Run the DSA Main.java test
7. Start PostgreSQL Docker container
8. Run the Spring Boot CRUD app
```

Once your Java + Spring Boot app runs successfully, tell me:

```text
“Setup done”
```

Then we’ll continue with **vetted Java + Spring Boot interview questions**, this time properly: actual questions, model answers, follow-ups, traps, and code.
