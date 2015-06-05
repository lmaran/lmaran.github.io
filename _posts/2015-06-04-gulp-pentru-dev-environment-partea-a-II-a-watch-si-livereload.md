---
layout: post
title:  "Gulp pentru dev environment. Partea a II-a - Watch si Livereload"
date:   2015-06-04 00:00:01
comments: true
categories: DevOps
---

In contextul dezvoltarii unei aplicatii web bazata pe javascript (Node.js), termenul de `Watch` se refera la monitorizarea fisierelor (client sau server-side), iar `Livereload` se refera la reincarcarea automata a paginii in browser in momentul in care `watch`-ul detecteaza o modificare.

Daca vrei sa sari peste explicatii, poti accesa direct codul sursa de [aici](https://github.com/lmaran/Gulp-starter/blob/master/gulpfile.js).

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/06-05-gulp_watch_and_livereload.png)

Asa cum reiese si din figura, monitorizarea client-side se realizeaza cu [gulp.watch()](https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpwatchglob--opts-tasks-or-gulpwatchglob--opts-cb) iar cea server-side cu [nodemon](https://github.com/remy/nodemon). In ambele cazuri, in momentul in care se detecteaza o modificare in codul sursa, se trimite o notificare server-ului de [Livereload](https://github.com/vohof/gulp-livereload) care, mai departe, va comanda un refresh in browser.


### Livereload server ###

`Livereload.listen()` are rol de server (hub) care centralizeaza cererile primite de la diversi clienti (nodemon / gulp.watch) si le transmite, mai departe, unei componente care ruleaza in browser.

Pentru rularea server-ului de `Livereload` am creat un task cu o singura linie:

```js
gulp.task('livereload', function() {
    livereload.listen();
});
```

Cele mai importante comenzi la care ascula acest server sunt:

- `livereload.changed(path)` - comanda reincarcarea unei anumite resurse dintr-o pagina (ex: reincarca doar fisierul css modificat). 
 - `pipe(livereload())` - este o comanda echivalenta, cu exceptia faptului ca se aplica pe un stream de Gulp
- `livereload.reload(file)` - comanda reincarcarea unei intregi pagini (ex: atunci cand se adauga/redenumeste/sterge un fisier js/css definit in index.html, atunci intreg fisierul index.html trebuie reincarcat.

### Livereload browser plugin ###

Reincarcarea efectiva a unei pagini (sau a unei portiuni dintr-o pagina) se realizeaza cu ajutorul unei componente care ruleaza in browser. Pentru acesta componenta ai 2 optiuni:

1. instalezi un [plugin](http://livereload.com/extensions/) direct in browser
2. instalezi un middleware ([connect-livereload](https://github.com/intesso/connect-livereload)) in codul care ruleaza partea de server (Node.js)

Eu am preferat sa nu depind de un anumit plugin asa ca, in pagina de start (app.js) am adaugat urmatoarea secventa:

```js
var app = express();

if (app.get('env') == 'development') {
    app.use(require('connect-livereload')());
};
```

### Monitorizarea client-side (gulp.watch) ###

```js
gulp.task('watch', function() {

    function injectJsAndReload(cb){
        gulp.src('./client/index.html')
            .pipe(inject(gulp.src('./client/app/**/*.js', {read: false}), {relative: true})) // js app files   
            .pipe(gulp.dest('./client/'))
            .on('end', function(){ 
                livereload.reload();
            }); 
    };
        
    function injectCssAndReload(){
        gulp.src('./client/index.html')
            .pipe(inject(gulp.src('./client/app/**/*.css', {read: false}), {relative: true})) // css app files   
            .pipe(gulp.dest('./client/'))
            .on('end', function(){ 
                livereload.reload();
            }); 
    };

    gulp.watch('client/app/**/*.*').on('change', function(file) { // no "./" in front of glob       
        var ext = path.extname(file.path); // ex: .js       
        var fileName = path.basename(file.path); // ex: test.css
        var crtDir = path.parse(file.path).dir; // ex: c:/.../controllers
        var name = path.parse(file.path).name; // ex: test      

        gutil.log(gutil.colors.cyan('watch-all'), 'saw',  gutil.colors.magenta(fileName), 'was ' + file.type); 
        
        if(ext == '.less'){                                                 
            if(file.type === 'deleted'){
                del(path.join(crtDir, name) + '.css'); 
            } else if(file.type === 'added' || file.type === 'changed' || file.type === 'renamed'){
                gulp.src(file.path)
                    .pipe(less())
                    .pipe(gulp.dest(path.parse(file.path).dir)); 
            };             
        
        } else if(ext == '.css'){                  
            if(file.type === 'added' || file.type === 'renamed'){
                injectCssAndReload();
            } else if(file.type === 'deleted'){ 
                del(file.path, injectCssAndReload);
            } else if(file.type === 'changed'){
                livereload.changed(fileName);         
            };             
        
        } else if(ext == '.js'){                
            if(file.type === 'added' || file.type === 'renamed'){
                injectJsAndReload();
            } else if(file.type === 'deleted'){
                del(file.path, injectJsAndReload); 
            } else if(file.type === 'changed'){
                gulp.src(file.path)    
                    .pipe(jshint())
                    .pipe(jshint.reporter(stylish))
                    .pipe(jshint.reporter('fail'))
                    .on('error', function(error){
                        gutil.log(gutil.colors.red('JSHINT failed!'));
                        this.emit('end'); // end the current task so that 'passed' msg is no longer displayed
                    })                                   
                    .pipe(notify(function(){ 
                        gutil.log(gutil.colors.green('JSHINT passed!'));
                        livereload.changed(fileName);
                    }));
            };         
        };
    });
});
```

Explicatii:

- `injectJsAndReload()` si `injectCssAndReload()` sunt 2 functii care scaneaza toate fisierele `js` respectiv `css` din aplicatie si le injecteaza in `index.html` (vezi placeholder-ul `<--inject-->`). La final se declanseaza un full page reload - asta deoarece se presupune ca in index.html au aparut modificari la referintele catre anumite fisiere js/css.
- am folosit `pipe.notify()` ca si un wrapper care imi permite sa rulez o secventa de cod javascript in interiorul unui pipe()
- a trebuit sa monitorizez toate fisierele, in toate directoarele: `gulp.watch('client/app/**/*.*')`. Am observat ca daca configurez watch-uri separate pentru cele 3 tipuri de fisiere (ex: `gulp.watch('client/app/**/*.js')`), atunci exista cel putin 2 situatii in care monitorizarea nu functioneaza:

 - daca adaug un fisier intr-un folder nou creat
 - daca adaug un fisier intr-un folder care, initial nu continea niciun fisier de tipul celui monitorizat 

### Monitorizarea server-side (nodemon) ###

Pentru partea de server nu este suficient un refresh in browser. [Nodemon](https://github.com/remy/nodemon) este un modul care monitorizeaza orice schimbare aparuta pe partea de server-side (node.js) si restarteaza automat procesul aferent aplicatiei. 

```js
gulp.task('watch-server', function() {
	nodemon({
    		script: 'server/app.js',
    		ext: 'js html',
            env: {'NODE_ENV':'development' },
            ignore: ['node_modules/', 'client', 'gulpfile.js']
        })
	   .on('restart', function(){                 
            gulp.src('server/app.js', {read:false})
                .pipe(livereload()); 
        });
});
```

### Concluzie ###

Codul poate parea putin cam lung (in comparatie cu alte surse disponibile pe Internet) dar am preferat sa ma concentrez pe eficienta cu care sunt prelucrate task-urile si mai putin pe dimensiunea codului. Spre exemplu:

- daca modific un fisier `.js`, doar acest fisier este analizat cu [jshint](https://github.com/spalger/gulp-jshint) (nu toate)
- daca modific un fisier `.less`, doar acest fisier este [compilat](https://github.com/plus3network/gulp-less) in .css (nu toate)

Mai mult, datorila lui [livereload](https://github.com/vohof/gulp-livereload), orice fisier modific, pe client sau server-side, modificarile se reflecta automat in browser. 