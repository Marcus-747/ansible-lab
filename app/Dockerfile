FROM nginx:stable-alpine 
ARG BUILD_NUMBER=dev 
COPY index.html /usr/share/nginx/html/index.html 
RUN sed -i "s/__BUILD__/${BUILD_NUMBER}/" /usr/share/nginx/html/index.html 
EXPOSE 80
