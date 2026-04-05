**Αυτοματοποίηση και Containerization (DevOps Approach)** 



*# Χρήση επίσημου OpenJDK image (Java 17)*

*FROM openjdk:17-jdk-alpine*

*# Ορισμός μεταβλητής για το αρχείο .jar της εφαρμογής ARG*

*JAR\_FILE=target/insurance-portal-1.0.0.jar*

*# Αντιγραφή του jar στο container*

*COPY ${JAR\_FILE} app.jar*

*# Άνοιγμα της θύρας 8080*

*33*

*EXPOSE 8080*

*# Εντολή εκκίνησης της εφαρμογής*

*ENTRYPOINT \["java","-jar","/app.jar"]* 

