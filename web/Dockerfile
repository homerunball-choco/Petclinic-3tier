
# syntax=docker/dockerfile:1

FROM httpd:2.4-alpine

# Reverse Proxy 모듈 활성화
RUN sed -i \
    -e 's/^#LoadModule proxy_module /LoadModule proxy_module /' \
    -e 's/^#LoadModule proxy_http_module /LoadModule proxy_http_module /' \
    /usr/local/apache2/conf/httpd.conf && \
    printf '\nInclude conf/extra/petclinic.conf\n' \
    >> /usr/local/apache2/conf/httpd.conf

# Apache 프록시 설정
COPY docker/petclinic.conf /usr/local/apache2/conf/extra/petclinic.conf

# 미용 예약 WEB 페이지
COPY index.html /usr/local/apache2/htdocs/index.html

EXPOSE 80

CMD ["httpd-foreground"]
