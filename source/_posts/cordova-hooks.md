---
title: hooks功能-檢查、壓縮、合併
date: 2016-12-23 10:51:17
categories:
- APP
- Cordova
tags:
- Cordova
---

最近才得知cordova可以在編譯輸出前做一些事情
這個地方叫做hooks，可以到[官網](https://cordova.apache.org/docs/en/latest/guide/appdev/hooks/)查詢

預計要來做幾件事情
1.JSHint
2.Concat files
3.Uglify

## JSHint

檢查JS語法，這個會放在before_prepare，要讓錯誤扼殺在成長之前XD
所以我們要建立一個檔案，`010_jshint.js`

hooks的運作機制，會根據js前面的號碼作為排序，來依序執行檔案
有多個動作時，記得要確認一下要執行的順序為何

```bash
#!/usr/bin/env node
```

```js
var fs = require('fs');
var path = require('path');
var jshint = require('jshint').JSHINT;
var async = require('async');

var recursiveFolderSearch = true;
var foldersToProcess = [
  'js'
];

foldersToProcess.forEach(function (folder) {
  processFiles("www/" + folder);
});

function processFiles(dir, callback) {
  var errorCount = 0;
  fs.readdir(dir, function (err, list) {
    if (err) {
      console.error('Directory Error: ' + err);
      return;
    }
    async.eachSeries(list, function (file, innercallback) {
      file = dir + '/' + file;
      fs.stat(file, function (err, stat) {
        if (!stat.isDirectory()) {
          if (path.extname(file) === ".js") {
            lintFile(file, function (hasError) {
              if (hasError) {
                errorCount++;
              }
              innercallback();
            });
          } else {
            innercallback();
          }
        } else {
          if (stat.isDirectory() && recursiveFolderSearch) {
            processFiles(file, callback);
          } else {
            innercallback();
          }
          innercallback();
        }
      });
    }, function (error) {
      if (errorCount > 0) {
        console.error('Get ' + errorCount + ' error(s) by JSHint.');
        process.exit(1);
      }
    });
  });
}

function lintFile(file, callback) {
  console.log("Linting " + file);
  fs.readFile(file, function (err, data) {
    if (err) {
      console.log('Error: ' + err);
      return;
    }
    if (jshint(data.toString())) {
      console.log('File ' + file + ' has no errors.');
      console.log('-----------------------------------------');
      callback(false);
    } else {
      console.log('Errors in file ' + file);
      var out = jshint.data(),
        errors = out.errors;
      for (var j = 0; j < errors.length; j++) {
        console.log(errors[j].line + ':' + errors[j].character + ' -> ' + errors[j].reason + ' -> ' +
          errors[j].evidence);
      }
      console.log('-----------------------------------------');
      callback(true);
    }
  });
}
```

## Concat files

撰寫時，我們會將js、css根據模組或功能等需求而分開來
等到我們要執行時，為了效能的優化，就會將檔案進行合併

這個動作會放在after_prepare下，這邊也要建立一個檔案`012_concat_files_by_index.js`
這功能主要是將index.html中所宣告引用的js、css進行合併，所以index.html還是要記得先加上

```bash
#!/usr/bin/env node
```

```js
// Parses the index.html file search for js and css files.
// v1.0
// all the files will be concatenated in 2 files.
// Note: It does not touch cordova.js.
// appConcatFolder: folder for the js bundle.
// appConcatFile: name of the js file bundle.
// cssConcatFolder = folder of the css bundle;
// cssConcatFile = name of the css file bundle;

var fs = require('fs');
var path = require('path');
var htmlparser = require("htmlparser2");
var Concat = require('concat-with-sourcemaps');

var rootDir = process.argv[2];
var platformPath = path.join(rootDir, 'platforms');
var platform = process.env.CORDOVA_PLATFORMS;
var cliCommand = process.env.CORDOVA_CMDLINE;

// hook configuration
var isRelease = (cliCommand.indexOf('--release') > -1) || (cliCommand.indexOf('--runrelease') > -1);
// isRelease = true;

if (!isRelease) {
  return;
}

switch (platform) {
  case 'android':
    platformPath = path.join(platformPath, platform, 'assets', 'www');
    break;
  case 'ios':
    platformPath = path.join(platformPath, platform, 'www');
    break;
  default:
    console.log('this hook only supports android and ios currently');
    return;
}

console.log('************************************************************************************');
console.log('... Parsing index.html for references (js & css) and concatenation ...');
console.log('************************************************************************************');

var appConcatFolder = 'js';
var appConcatFile = 'app.min.js';

var cssConcatFolder = 'css';
var cssConcatFile = 'app.min.css';

// Creating folder if it does not exist.
if (!fs.existsSync(path.join(platformPath, appConcatFolder))) {
  fs.mkdirSync(path.join(platformPath, appConcatFolder));
}

if (!fs.existsSync(path.join(platformPath, cssConcatFolder))) {
  fs.mkdirSync(path.join(platformPath, cssConcatFolder));
}

// Processing index.html to file js file and css files.
var elements = processIndexHtml(path.join(platformPath, "index.html"));

// Removes all the scripts and css files referenced in the index.html
// Removes other folders.
if (elements) {
  if (elements.scripts.length > 0) {
    elements.scripts.forEach(function (file) {
      console.log('Removing referenced file: ' + path.join(platformPath, file));
      fs.unlink(path.join(platformPath, file),
        function (err) {
          if (err) {
            console.log(err);
            throw err;
          }
        });
    });
  }
  if (elements.css.length > 0) {
    elements.css.forEach(function (file) {
      console.log('Removing referenced file: ' + path.join(platformPath, file));
      fs.unlink(path.join(platformPath, file),
        function (err) {
          if (err) {
            console.log(err);
            throw err;
          }
        });
    });
  }
}

// Read the index.html file and extracts all the references to scripts and css.
// compacts js in one file
// compacts css in one file.
function processIndexHtml(file) {
  var html = readIndexHtml(file);
  if (!html) {
    return null;
  }
  var elements = parseIndexHtml(html);
  if (!elements) {
    return null;
  }

  if (elements.scripts.length > 0) {
    var concatJs = new Concat(false, appConcatFile, '\n');
    elements.scripts.forEach(function (file) {
      console.log('Processing file for concatenation : ' + path.join(platformPath, file));
      var fileContent = String(fs.readFileSync(path.join(platformPath, file)));
      concatJs.add(file, fileContent);
    });
    fs.writeFileSync(path.join(platformPath, appConcatFolder + '/' + appConcatFile), concatJs.content);
  } else {
    console.log('No js scripts to process.');
  }
  if (elements.css.length > 0) {
    var concatCss = new Concat(false, cssConcatFile, '\n');
    elements.css.forEach(function (file) {
      console.log('Processing file for concatenation : ' + path.join(platformPath, file));
      var fileContent = String(fs.readFileSync(path.join(platformPath, file)));
      concatCss.add(file, fileContent);
    });
    fs.writeFileSync(path.join(platformPath, cssConcatFolder + '/' + cssConcatFile), concatCss.content);
  } else {
    console.log('No css scripts to process.');
  }

  return elements;
}

// Parse index.html file to get all the references to js scripts and css files.
function parseIndexHtml(html) {
  var scripts = [];
  var css = [];

  var parser = new htmlparser.Parser({
    onopentag: function (name, attribs) {
      console.log(JSON.stringify(attribs));
      if (name === "script" && attribs.src.startsWith('js/')) {
        scripts.push(attribs.src);
      } else if (name === "link" && attribs.rel === "stylesheet") {
        css.push(attribs.href);
      }
    },
    ontext: function (text) {},
    onclosetag: function (tagname) {}
  }, {
    decodeEntities: true
  });

  parser.write(html);
  parser.end();

  return {
    scripts: scripts,
    css: css
  };
}

function readIndexHtml(file) {
  // var opts = {};
  try {
    var html = String(fs.readFileSync(file));
    return html;
  } catch (err) {
    console.log('readIndexHtml => Error', err);
    return "";
  }
}
```

做完這段以後可以發現到，index.html中已經被合併的檔案tag還是存在，那接著我們就要對於這些被合併的tag換成最後輸出的檔案
那這邊需要先對index.html中加上一些東西，才能讓接下來的功能順利進行，可以參考[node-useref](https://github.com/digisfera/useref)

```html
<!-- build:css css/app.min.css -->
<link href="css/ionic.app.css" rel="stylesheet" />
<link href="css/style.css" rel="stylesheet" />
<!-- endbuild -->
<!-- build:js js/app.min.js -->
<script src="js/app.js"></script>
<!-- endbuild -->
```
一樣在after_prepare中，加上015_prepare_index.js

```bash
#!/usr/bin/env node
```

```js
// useref index.html
// v1.0
// searches the node-useref tags in your index.html to replace it width
// the bundle.
// https://www.npmjs.com/package/node-useref

var fs = require('fs');
var path = require('path');
var useref = require('node-useref');

var rootDir = process.argv[2];
var platformPath = path.join(rootDir, 'platforms');
var platform = process.env.CORDOVA_PLATFORMS;
var cliCommand = process.env.CORDOVA_CMDLINE;

// hook configuration
var isRelease = (cliCommand.indexOf('--release') > -1) || (cliCommand.indexOf('--runrelease') > -1);
// isRelease = true;

if (!isRelease) {
  return;
}

switch (platform) {
  case 'android':
    platformPath = path.join(platformPath, platform, 'assets', 'www');
    break;
  case 'ios':
    platformPath = path.join(platformPath, platform, 'www');
    break;
  default:
    console.log('this hook only supports android and ios currently');
    return;
}

console.log('************************************************************************************');
console.log('... useref index.html ...');
console.log('************************************************************************************');

userefIndexHtml(path.join(platformPath, "index.html"));

function userefIndexHtml(file) {
  var opts = {};

  try {
    var res = fs.readFileSync(file, {
      encoding: 'utf-8'
    });
    var output = useref(res, opts);
    var html = output[0];

    fs.writeFileSync(file, html);

  } catch (err) {
    console.log('userefIndexHtml: Error', err);
  }
}
```

## Uglify

這是裡面最簡單的好做到的功能XD，只要下個指令就可以
會自動在after_prepare下增加一個`uglify.js`檔案，因為前面沒有順序，所以會是最後執行
詳細內容一樣可以到[官網](https://github.com/rossmartin/cordova-uglify)中查詢

``` bash
npm install cordova-uglify --save
```

做完以上的動作，我們就可以下個指令確認一下

``` bash
cordova prepare [platform] --release
```

沒有看到任何錯誤就是成功啦
可以打完收工囉

ㄟㄟㄟ～等等！
mac用戶明明會遇到另一個錯誤

好吧～我只好再回來繼續
因為mac用戶會有權限的問題，會只看到一個鬼一般的訊息
>Error: EACCES

那這個問題不大，只要下個指令給個權限就好，針對我們在after_prepare所新增的兩個檔案012、015

``` bash
chmod +x hooks/after_prepare/[filename]
```
好啦，這次真的要打完收工了