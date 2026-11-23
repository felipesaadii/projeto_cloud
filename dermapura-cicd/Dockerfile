# Imagem leve do Nginx para servir a One-Page estática
FROM nginx:alpine

# Remove a página padrão do Nginx e publica a página DermaPura
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html

# Porta HTTP do servidor web
EXPOSE 80

# Nginx permanece em primeiro plano dentro do container
CMD ["nginx", "-g", "daemon off;"]
