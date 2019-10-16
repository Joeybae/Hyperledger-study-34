# Hyperledger-study-34

하이퍼레져 앱 만들기 실습 34일차 - 객체, 모듈, API

출처 : https://opentutorials.org/course/3332/21153

1. 데이터와 값을 담는 그릇으로서 객체 - syntax/object3.js

        var v1 = 'v1';
        v1 = 'egoing';
        var v1 = 'v2';

        var q = {
            v1 : 'v1',
            v2 : 'v2',
            f1 : function (){
                console.log(this.v1);
            },
            f2: function (){
                console.log(this.v2);
            }
        }

        q.f1();
        q.f2();

2. 객체를 이용하여 정리 - lib/template.js

        var template = {
          HTML:function(title, list, body, control){
            return `
            <!doctype html>
            <html>
            <head>
              <title>WEB1 - ${title}</title>
              <meta charset="utf-8">
            </head>
            <body>
              <h1><a href="/">WEB</a></h1>
              ${list}
              ${control}
              ${body}
            </body>
            </html>
            `;
          },list:function(filelist){
            var list = '<ul>';
            var i = 0;
            while(i < filelist.length){
              list = list + `<li><a href="/?id=${filelist[i]}">${filelist[i]}</a></li>`;
              i = i + 1;
            }
            list = list+'</ul>';
            return list;
          }
        }

3. module

nodejs/mpart.js

      var M = {
          v:'v',
          f:function(){
              console.log(this.v);
          }
      }

      module.exports = M;

nodejs/muse.js

      var part = require('./mpart.js');
      part.f();

4. module 활용하여 정리

lib/template.js

      module.exports = {
          HTML:function (title, list, body, control){
            return `
            <!doctype html>
            <html>
            <head>
              <title>WEB1 - ${title}</title>
              <meta charset="utf-8">
            </head>
            <body>
              <h1><a href="/">WEB</a></h1>
              ${list}
              ${control}
              ${body}
            </body>
            </html>
            `;
          },
          list:function (filelist){
            var list = '<ul>';
            var i = 0;
            while(i < filelist.length){
              list = list + `<li><a href="/?id=${filelist[i]}">${filelist[i]}</a></li>`;
              i = i + 1;
            }
            list = list+'</ul>';
            return list;
          }
        }

main.js

        var http = require('http');
        var fs = require('fs');
        var url = require('url');
        var qs = require('querystring');
        var template = require('./lib/template.js');
        var path = require('path');
        var sanitizeHtml = require('sanitize-html');

        var app = http.createServer(function(request,response){
            var _url = request.url;
            var queryData = url.parse(_url, true).query;
            var pathname = url.parse(_url, true).pathname;
            if(pathname === '/'){
              if(queryData.id === undefined){
                fs.readdir('./data', function(error, filelist){
                  var title = 'Welcome';
                  var description = 'Hello, Node.js';
                  var list = template.list(filelist);
                  var html = template.HTML(title, list,
                    `<h2>${title}</h2>${description}`,
                    `<a href="/create">create</a>`
                  );
                  response.writeHead(200);
                  response.end(html);
                });
              } else {
                fs.readdir('./data', function(error, filelist){
                  var filteredId = path.parse(queryData.id).base;
                  fs.readFile(`data/${filteredId}`, 'utf8', function(err, description){
                    var title = queryData.id;
                    var sanitizedTitle = sanitizeHtml(title);
                    var sanitizedDescription = sanitizeHtml(description, {
                      allowedTags:['h1']
                    });
                    var list = template.list(filelist);
                    var html = template.HTML(sanitizedTitle, list,
                      `<h2>${sanitizedTitle}</h2>${sanitizedDescription}`,
                      ` <a href="/create">create</a>
                        <a href="/update?id=${sanitizedTitle}">update</a>
                        <form action="delete_process" method="post">
                          <input type="hidden" name="id" value="${sanitizedTitle}">
                          <input type="submit" value="delete">
                        </form>`
                    );
                    response.writeHead(200);
                    response.end(html);
                  });
                });
              }
            } else if(pathname === '/create'){
              fs.readdir('./data', function(error, filelist){
                var title = 'WEB - create';
                var list = template.list(filelist);
                var html = template.HTML(title, list, `
                  <form action="/create_process" method="post">
                    <p><input type="text" name="title" placeholder="title"></p>
                    <p>
                      <textarea name="description" placeholder="description"></textarea>
                    </p>
                    <p>
                      <input type="submit">
                    </p>
                  </form>
                `, '');
                response.writeHead(200);
                response.end(html);
              });
            } else if(pathname === '/create_process'){
              var body = '';
              request.on('data', function(data){
                  body = body + data;
              });
              request.on('end', function(){
                  var post = qs.parse(body);
                  var title = post.title;
                  var description = post.description;
                  fs.writeFile(`data/${title}`, description, 'utf8', function(err){
                    response.writeHead(302, {Location: `/?id=${title}`});
                    response.end();
                  })
              });
            } else if(pathname === '/update'){
              fs.readdir('./data', function(error, filelist){
                var filteredId = path.parse(queryData.id).base;
                fs.readFile(`data/${filteredId}`, 'utf8', function(err, description){
                  var title = queryData.id;
                  var list = template.list(filelist);
                  var html = template.HTML(title, list,
                    `
                    <form action="/update_process" method="post">
                      <input type="hidden" name="id" value="${title}">
                      <p><input type="text" name="title" placeholder="title" value="${title}"></p>
                      <p>
                        <textarea name="description" placeholder="description">${description}</textarea>
                      </p>
                      <p>
                        <input type="submit">
                      </p>
                    </form>
                    `,
                    `<a href="/create">create</a> <a href="/update?id=${title}">update</a>`
                  );
                  response.writeHead(200);
                  response.end(html);
                });
              });
            } else if(pathname === '/update_process'){
              var body = '';
              request.on('data', function(data){
                  body = body + data;
              });
              request.on('end', function(){
                  var post = qs.parse(body);
                  var id = post.id;
                  var title = post.title;
                  var description = post.description;
                  fs.rename(`data/${id}`, `data/${title}`, function(error){
                    fs.writeFile(`data/${title}`, description, 'utf8', function(err){
                      response.writeHead(302, {Location: `/?id=${title}`});
                      response.end();
                    })
                  });
              });
            } else if(pathname === '/delete_process'){
              var body = '';
              request.on('data', function(data){
                  body = body + data;
              });
              request.on('end', function(){
                  var post = qs.parse(body);
                  var id = post.id;
                  var filteredId = path.parse(id).base;
                  fs.unlink(`data/${filteredId}`, function(error){
                    response.writeHead(302, {Location: `/`});
                    response.end();
                  })
              });
            } else {
              response.writeHead(404);
              response.end('Not found');
            }
        });
        app.listen(3000);

5. 입력 정보 보안 - main.js

        fs.readdir('./data', function(error, filelist){
                var filteredId = path.parse(queryData.id).base;
                fs.readFile(`data/${filteredId}`, 'utf8', function(err, description){
                  var title = queryData.id;
                  var list = template.list(filelist);
                  var html = template.HTML(title, list,
                    `
                    <form action="/update_process" method="post">
                      <input type="hidden" name="id" value="${title}">
                      <p><input type="text" name="title" placeholder="title" value="${title}"></p>
                      <p>
                        <textarea name="description" placeholder="description">${description}</textarea>
                      </p>
                      <p>
                        <input type="submit">
                      </p>
                    </form>
                    `,
                    `<a href="/create">create</a> <a href="/update?id=${title}">update</a>`
                  );
                  response.writeHead(200);
                  response.end(html);
                });
              });

6. 출력 정보 보안 - main.js

  1) package.json 생성

    # npm init

  2) sanitize module 설치

    # npm install -S sanitize-html
    
  3) title, description 보안파일     
    
    var sanitizedTitle = sanitizeHtml(title);
                var sanitizedDescription = sanitizeHtml(description, {
                  allowedTags:['h1']
                });
