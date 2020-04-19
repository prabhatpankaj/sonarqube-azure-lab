FROM sonarqube:7.0-alpine


ENV SONAR_KOTLIN_COMMUNITY_VERSION=0.5.0 \
    SONAR_AUTH_AAD_VERSION=1.0 \
    SONAR_JAVA_VERSION=5.7.0.15470 \
    SONAR_CSHARP_VERSION=7.10.0.7896

RUN wget "https://github.com/SonarSource/sonar-dotnet/releases/download/${SONAR_CSHARP_VERSION}/sonar-csharp-plugin-${SONAR_CSHARP_VERSION}.jar" \
    && wget "https://github.com/SonarQubeCommunity/sonar-auth-aad/releases/download/${SONAR_AUTH_AAD_VERSION}/sonar-auth-aad-plugin-${SONAR_AUTH_AAD_VERSION}.jar" \
    && wget "https://dl.bintray.com/sonarsource/SonarQube/org/sonarsource/java/sonar-java-plugin/${SONAR_JAVA_VERSION}/sonar-java-plugin-${SONAR_JAVA_VERSION}.jar" \
    && wget "https://github.com/arturbosch/sonar-kotlin/releases/download/${SONAR_KOTLIN_COMMUNITY_VERSION}/sonar-kotlin-${SONAR_KOTLIN_COMMUNITY_VERSION}.jar" \
    && mv *.jar $SONARQUBE_HOME/extensions/plugins \
    && ls -lah $SONARQUBE_HOME/extensions/plugins

# Configure Azure Web App database entrypoint
COPY entrypoint.sh ./bin/
RUN chmod +x ./bin/entrypoint.sh
ENTRYPOINT ["./bin/entrypoint.sh"]
