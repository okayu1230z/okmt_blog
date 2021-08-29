ARG DEPLOY_IMAGE=nginx:latest
FROM $DEPLOY_IMAGE
LABEL maintainer="okmt <yuki.okamoto@okmt.dev>"

ADD ./nginx.conf /etc/nginx/nginx.conf
CMD ["nginx", "-g", "daemon off;"]
