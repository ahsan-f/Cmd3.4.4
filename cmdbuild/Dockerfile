# Use the official Tomcat image with JDK 21
FROM tomcat:11.0.2-jdk21-temurin-noble

# Set the working directory inside the container
WORKDIR $CATALINA_HOME

# Set environment variables for the application configuration
ENV CMDBUILD_URL https://sourceforge.net/projects/cmdbuild/files/latest/download
ENV POSTGRES_USER postgres
ENV POSTGRES_PASS postgres
ENV POSTGRES_PORT 5432
ENV POSTGRES_HOST cmdbuild_db
ENV POSTGRES_DB cmdbuild_db3
ENV CMDBUILD_DUMP demo

# Install necessary packages in a single, efficient RUN command
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    maven \
    postgresql-client \
    zip unzip \
    libssl3 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create application-specific directories
RUN mkdir -p $CATALINA_HOME/conf/cmdbuild/ \
    && mkdir -p $CATALINA_HOME/webapps/cmdbuild/

# Copy configuration files and the entrypoint script from the build context
# The 'cmdbuild/files/' path is relative to the build context, which is the directory containing the Dockerfile.
COPY cmdbuild/files/tomcat-users.xml $CATALINA_HOME/conf/tomcat-users.xml
COPY cmdbuild/files/context.xml $CATALINA_HOME/webapps/manager/META-INF/context.xml
COPY cmdbuild/files/database.conf $CATALINA_HOME/conf/cmdbuild/database.conf
# COPY --chown=tomcat:tomcat --chmod=755 files/docker-entrypoint.sh /usr/local/bin/
COPY --chmod=755 --chown=tomcat:tomcat cmdbuild/files/docker-entrypoint.sh /usr/local/bin/

# Create the 'tomcat' user and group
RUN groupadd tomcat \
    && useradd -s /bin/false -g tomcat -d $CATALINA_HOME tomcat

# Download, unzip, and configure the CMDBuild application
RUN set -x \
    && wget -O /tmp/cmdbuild.war --no-check-certificate "$CMDBUILD_URL" \
    && unzip /tmp/cmdbuild.war -d $CATALINA_HOME/webapps/cmdbuild/ \
    && mv /tmp/cmdbuild.war $CATALINA_HOME/webapps/cmdbuild.war \
    && chmod 755 $CATALINA_HOME/webapps/cmdbuild/cmdbuild.sh \
    && chown -R tomcat:tomcat $CATALINA_HOME \
    && rm -rf /tmp/*

# The entrypoint script will be executed when the container starts
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

# Switch to the non-root 'tomcat' user for security
USER tomcat

# Expose the default Tomcat port
EXPOSE 8080

# The default command to run when the container starts
CMD ["catalina.sh", "run"]
