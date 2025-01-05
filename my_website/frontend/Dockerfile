# FROM nginx@sha256:2bcabc23b45489fb0885d69a06ba1d648aeda973fae7bb981bafbb884165e514 is the digest of the nginx:1.21 image. Use in production to ensure the same image is used (tags are mutable).

# Use the official nginx image
FROM nginx:1.21

# Remove the default static files 
RUN rm -rf /usr/share/nginx/html/

# Copy the static files from the static directory to the nginx directory
COPY static/ /usr/share/nginx/html/
