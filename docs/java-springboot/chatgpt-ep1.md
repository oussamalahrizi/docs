# 📘 Java + Spring Boot Journey: Terminal-First Dev Setup

## 🚀 Why I'm Avoiding IntelliJ (For Now)

While IntelliJ IDEA is powerful, I’ve chosen to stick with **VS Code + Terminal** to:

* Strengthen my understanding of **Java fundamentals**
* Learn how to use **Maven** properly
* Understand **project structure and classpaths**
* Avoid depending on IDE magic and auto-configs

---

## 🧠 What I'm Gaining by Using Terminal + VS Code

### ✅ Build Tool Mastery

* Learning **Maven goals**:

  * `clean`, `compile`, `test`, `package`, `install`, `spring-boot:run`
* Using the **Maven lifecycle**:

  ```
  validate → compile → test → package → verify → install → deploy
  ```
* Using `./mvnw` for consistent builds across machines

### ✅ Folder Structure Awareness

* `src/main/java` – Application source code
* `src/main/resources` – Configuration files (`application.yml` or `.properties`)
* `src/test/java` – Test cases
* `target/` – Build output (JARs, classes, etc.)

### ✅ Debugging Without IDE Dependency

* Running the app manually:

  ```bash
  mvn spring-boot:run
  ```
* Building the app:

  ```bash
  mvn clean install
  ```
* Testing:

  ```bash
  mvn test
  ```
* Optional debug flags:

  ```bash
  java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=*:5005 -jar target/app.jar
  ```

---

## 🚰 Recommended VS Code Setup

### Extensions

* Java Extension Pack (Microsoft)
* Spring Boot Extension Pack
* Debugger for Java
* Maven for Java
* Lombok support
* Vim emulation (optional)

### VS Code Settings

```json
"java.configuration.updateBuildConfiguration": "automatic",
"java.errors.incompleteClasspath.severity": "ignore"
```

---

## 🧙 Stewie’s Take (Yes, the Family Guy One)

> “Oh brilliant! Someone who doesn’t just want IntelliJ to spoon-feed them like a pampered Java baby. You, my friend, are the *terminal wizard* Spring Boot needs. Carry on.”

---

## 📝 Summary

| Goal                    | Why It Matters                                    |
| ----------------------- | ------------------------------------------------- |
| Avoiding IntelliJ       | Build real understanding, not dependency          |
| Learning Maven          | Automate builds and understand the Java ecosystem |
| Using Terminal          | Portable, scriptable, DevOps-friendly             |
| Mastering Folder Layout | Makes debugging and structuring easier            |
| Staying in VS Code      | Lightweight, fast, and customizable               |

---

