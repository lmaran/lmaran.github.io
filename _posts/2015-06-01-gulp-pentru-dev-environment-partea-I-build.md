---
layout: post
title:  "Gulp pentru dev environment. Partea I - Build"
date:   2015-06-01 00:00:01
comments: true
categories: DevOps
---

De ceva vreme, cel putin pentru proiectele "de acasa", mi-am propus sa evit clasicul VisualStudio. Am incercat pe rand cu Sublime, Brackets si, in final, am ramas la [VisualStudio Code](https://code.visualstudio.com/). Procesul de **build** sunt nevoit acum sa-l implementez de la zero, iar pentru asta am ales sa ma folosesc de [Gulp](http://gulpjs.com/). Cand spun **build** pe dev. environment ma gandesc la 2 componente:

- procesul de `build` propriu-zis, care se executa la pornire:
   - compileaza less -> css
   - compileaza ts -> js (sa zicem)
   - injecteaza automat fisierele js/css in "index.html"
   - analizeaza calitatea codului
- si un `watcher`, care monitorizeaza fisierele editate si aplica operatiile de mai sus "on-the-fly"

Cele 2 componente le-am evidentiat in imaginea de mai jos:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/06-01-gulp-build-and-watch.png)

Am sa sar peste explicatiile de tipul "de ce Gulp si nu Grunt", cum se instaleaza si cum se configureaza Grunt, sau care sunt pachetele folosite in `gulpfile.js` si cum se initializeaza acestea.

In caz ca doresti sa sari pesti explicatii, poti accesa direct [codul sursa](https://github.com/lmaran/Gulp-starter/blob/master/gulpfile.js).

## Initializarea task-urilor

Task-ul pentru `build` (apelabil cu `gulp dev`):

```js
gulp.task('dev', function(cb) {
    runSequence(
        'clean-css',
        'less',
        'build-dev-html',
        'jshint',
    cb);
});
```

iar task-ul pentru `build` + `watch` (apelabil cu `gulp dev:watch`):

```js
gulp.task('dev:watch', function(cb) {
    runSequence(
        'dev',
        ['livereload', 'watch'],
    cb);
});
```

In ambele cazuri am folosit [runSequence](https://github.com/OverZealous/run-sequence) pentru a stabili care task-uri se executa in paralel (ex: `livereload` si `watch`) si care in serie (restul).

Al 2-lea task l-am declarat ca fiind implicit (apelabil cu `gulp`):

```js
gulp.task('default',['dev:watch']);
```

## Implementarea task-urilor

### clean-css ###

Vreau sa ma asigur ca nu au ramas, din greseala, fisiere `.css` care sa nu aibe un corespondent `.less'.

```js
gulp.task('clean-css', function (cb) {
    return del(['./client/app/**/*.css'], cb);
});
```

### less ###

Compileaza fisierele `.less` in `.css`. Fisierele (css) rezultate se genereaza in acelasi director ca si fisierele (less) sursa.

```js
gulp.task('less', function() {
    return gulp.src('./client/app/**/*.less')
        .pipe(less())
        .pipe(gulp.dest('./client/app'));
});
```

### build-html

Initial fisierul `index.html` nu contine referinte catre fisierele `js` sau `css` ci doar niste placeholder-e care indica locatia in care aceste referinte urmeaza a fi injectate:

```html
<!doctype html>
<html ng-app=app>
<head>
    <base href="/" />   
    <!-- cdn -->
    <link rel="stylesheet" type="text/css" href="//netdna.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" />
    
    <!-- inject-vendor:css -->
    <!-- endinject -->

    <!-- inject:css -->
    <!-- endinject -->
</head>
<body>     
    <h2>Gulp Demo</h2>
    <div ng-class="demoClass" ng-controller="demoController">{{itemValue | json}}</div>
    <!-- cdn -->
    <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
    
    <!-- inject-vendor:js -->
    <!-- endinject -->

    <!-- inject:js -->
    <!-- endinject -->
</body>
</html>
```

Task-ul care populeaza aceste tag-uri este:

```js
gulp.task('build-dev-html', function(){  
    return gulp.src('./client/index.html')
        .pipe(inject(gulp.src('./client/app/**/*.css', {read:false}), {relative: true})) // css app files  
        .pipe(inject(gulp.src('./client/app/**/*.js', {read: false}), {relative: true})) // js app files  
        .pipe(inject(gulp.src(bowerFiles(), {read: false}), {name:'inject-vendor', relative: true})) // bower js and css files  
        .pipe(gulp.dest('./client/'));
});
```

Unde:

- `{read:false}` - citeste doar numele, nu si continutul fisierului (mai eficient)
- `{relative:true}` - calea rezultata va fi relativa la `<base href="/" />` din index.html
- `bowerFiles()` - este un [plugin](https://github.com/ck86/main-bower-files) care genereaza o lista cu fisierele `.js` si `.css` pe baza definitiilor din fisierul `bower.json`

La sfarsitul acestei operatii, fisierul index.html va arata asa:

```html
<!doctype html>
<html ng-app=app>
<head>
    <base href="/" />   
    <!-- cdn -->
    <link rel="stylesheet" type="text/css" href="//netdna.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" />
    
    <!-- inject-vendor:css -->
    <link rel="stylesheet" href="bower_components/normalize.css/normalize.css">
    <!-- endinject -->

    <!-- inject:css -->
    <link rel="stylesheet" href="app/app.css">
	<link rel="stylesheet" href="app/components/component1/test.css">
    <!-- endinject -->
</head>
<body>    
    <h2>Node Demo22345</h2>
    <div ng-class="demoClass" ng-controller="demoController">{{itemValue | json}}</div>

    <!-- cdn -->
    <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
    
    <!-- inject-vendor:js -->
    <script src="bower_components/lodash/lodash.js"></script>
    <!-- endinject -->

    <!-- inject:js -->
    <script src="app/app.js"></script>
    <script src="app/controllers/demoController.js"></script>
    <script src="app/controllers/test.js"></script>
    <script src="app/directives/demoDirective.js"></script>
    <script src="app/services/demoService.js"></script>
    <!-- endinject -->
</body>
</html>
```

### jshint ###

- analizeaza toate fisierele `.js` din aplicatie, 
- raporteaza erorile folosind [stylish](https://github.com/sindresorhus/jshint-stylish) pe post de formater,
- genereaza o eroare daca intalneste un cod scris gresit
- eroarea este captata si tratata intr-un "event handler". Acesta, la randul lui, emite un eveniment de tip 'end' care opreste executia celorlalte pipe-uri din task-ul curent.
- plugin-ul [notify](https://github.com/mikaelbr/gulp-notify) l-am folosit pe post de wrapper (da, e un 'hack') care sa-mi permita sa rulez instructiuni javascript intr-un pipe.
- daca pana la fina nu apar erori, se generea un mesaj de succes. Vreau ca acest mesaj sa nu apara pentru fiecare fisier analizat ci o singura data - la sfarsit (`onLast: true`)

```js
gulp.task('jshint', function() { 
    return gulp.src('./client/app/**/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter(stylish))      
        .pipe(jshint.reporter('fail'))  
        .on('error', function(error){
            gutil.log(gutil.colors.red('JSHINT failed!'));
            this.emit('end');
        })
        .pipe(notify({
            onLast: true,
            message: function(){
                gutil.log(gutil.colors.green('JSHINT passed!'));
            }
        }));
});
```
Cum functioneaza `jshint`? Daca, spre exemplu, introduc in cod doua greseli:
 
![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-gulp-err-in-code.png)

atunci aceste erori le pot depista in procesul de build - faza de analiza a codului:

![](https://dl.dropboxusercontent.com/u/43065769/blog/images/2015/05-30-gulp-err-in-jshint.png)

Intr-un articol viitor am sa prezint cum mi-am configurat un watcher in Gulp astfel incat, de fiecare data cand editez, sterg sau adaug un fisier, procesul de build sa se reia automat. 