---
layout: post
title: "Lightweight Sinatra website + Octopress blog"
date: 2013-11-24 10:34:12 -0300
comments: true
categories:
- Webdev
author: Bruno Pazzim
---
<h3>Octopress</h3>
Você já ouviu falar do <a href="http://octopress.org/" target="_blank">Octopress</a>? Ele é um pequeno framework feito para o <a href="http://jekyllrb.com/" target="blank">Jekyll</a>, que gera de maneira estática um blog pronto para você sair usando. Apesar de enxuto, é bastante poderoso. Além de todo suporte para as principais redes sociais já imbutido, ele conta com diversos plugins diferentes, como áreas de código, gerador de categorias, exibição de gists do Github nos posts, e muitas outras funcionalidades bacanas.

<!-- more -->

Porém, não é feito para qualquer um. É um sistema de blog de hackers para hackers. No Octopress, não existe interface administrativa, como no WordPress, por exemplo. Todos os comandos vão ser rodados por você via linha de comando, e as configurações serão feitas editando alguns arquivos padrões gerados por ele. Até mesmo a criação de um novo post é feita pela linha de comando. Então saiba onde vocês está entrando.

<h3>Sinatra</h3>

<a href="http://www.sinatrarb.com/">Sinatra</a> é uma <a href="http://en.wikipedia.org/wiki/Domain-specific_language" target="_blank">DSL</a> criada para que rapidamente possamos criar aplicações web em Ruby com o mínimo de esforço. Bem menos poderoso que o Rails, serve bem para websites de duas páginas ou menos. Sempre costumo a fazer sites estáticos usando ele.

<h3>Juntando os dois</h3>

Recentemente me deparei com um pequeno problema ao querer colocar um blog gerado pelo Octopress em um subdiretório de uma aplicação rodando Sinatra (http://meuwebsite/blog, por exemplo). Resolvi criar um pequeno tutorial para que você também possa usufruir da combinação dessas duas ferramentes fantásticas, que vão manter seu diretório de trabalho limpo e deixar seu site bastante leve.

Primeiro, você precisa criar seu blog através do Octopress. Siga as instruções contidas <a href="http://octopress.org/docs/setup/" target="_blank">nessa página</a>.

Depois disso você já tem seu blog funcionando. Mas nós precisamos fazer algumas alterações nas configurações. Afinal, queremos que o nosso blog seja acessível através da página http://example.com/blog. Altere os seguintes arquivos que foram gerados:

``` ruby config.yml
    destination: public/blog
    url: http://example.com/blog
    subscribe_rss: /blog/atom.xml
    root: /blog
```

``` ruby config.rb - para Compass & Sass
    http_path = “/blog”
    http_images_path = “/blog/images”
    http_fonts_path = “/blog/fonts”
    css_dir = “public/blog/stylesheets”
```
``` ruby Rakefile.rb
    public_dir = “public/blog”
    # Se você está fazendo deploy com rsync, troque o caminho do Rakefile
    document_root = “~/yoursite.com/blog”
```

Uma vez feito isso, agora temos o blog no caminho que desejamos, /blog. Vá no terminal e execute o comando “rake generate preview”. Isso vai fazer com que o blog seja gerado e que um servidor seja iniciado para que você possa testar localmente. Vá até seu browser e tente acessar “localhost:4000/blog”. Você deve ver seu blog rodando.

<h3>Agora vamos criar nossa aplicação Sinatra.</h3>

Para começar nosso projeto, primeiro você deve instalar o Sinatra e o Thin, caso não o tenha feito ainda.

gem install sinatra<br>
gem install thin

Feito isso, vamos criar uma pasta onde nosso website vai ficar. Dentro dele, crie um arquivo chamado site.rb, com o seguinte conteúdo:

``` ruby site.rb
require ‘sinatra’

get ‘/’ do
  redirect ‘/index.html’
end
```

Como podem ver, estamos chamando um arquivo chamado index.html, que ainda não criamos. Dentro da raiz do nosso projeto, crie uma pasta chamada public, e dentro dela crie o index.html. Esse será o arquivo html do seu site enxuto, landing page, ou qualquer que seja a sua intenção. Se você rodar “ruby site.rb” no console, esse arquivo estará acessível pelo browser através do endereço “localhost:4567”.

<h3>A parte final: integrando o blog gerado com o aplicativo Sinatra que criamos.</h3>

Dentro da pasta onde geramos nosso blog, vá até a pasta /public e copie a pasta chamada blog. Cole ela em nosso projeto Sinatra, dentro da pasta /public. Feito isso, edite o arquivo site.rb e adicione essas linhas no final do arquivo:

```ruby site.rb
get(/.+/) do
  send_sinatra_file(request.path) {404}
end

not_found do
  send_file(File.join(File.dirname(__FILE__), ‘public’, ‘404.html’), {:status => 404})
end

def send_sinatra_file(path, &missing_file_block)
  file_path = File.join(File.dirname(__FILE__), ‘public’, path)
  file_path = File.join(file_path, ‘index.html’) unless file_path =~ /.[a-z]+$/i
  File.exist?(file_path) ? send_file(file_path) : missing_file_block.call
end
```

Vá ao terminal e dentro da pasta do projeto Sinatra execute “ruby site.rb”. Agora acesse através do seu browser a página “localhost:4567/blog” e você terá seu blog rodando em um subdiretório do seu projeto Sinatra. Se você entrar em “localhost:4567”, verá o arquivo HTML da sua página.

Criei um pequeno aplicativo para mostrar como funciona depois de pronto. Você pode acessá-lo no GitHub através <a href="https://github.com/brunopazzim/sinatra-octopress-demo" target="_blank">desse link</a>.
