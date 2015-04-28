
#使用Grunt来制作CSS sprite

>[Grunt][1]是前端的javascript自动化构建工具。[Grunt安装][2]

众所周知Grunt给前端带来的变化是从过去原始的自给自足的时代到现在自动化生产的工业时代，这样虽然说的有些夸张，但是自动化的检测语法，自动化的压缩，自动化的单元测试，都以一条命令行形式来解决，的确省了我们不少时间。下面用制作CSS sprite的例子来阐述Grunt安装和使用。

##建立项目
这主要是用node的package.json作为Grunt版本或插件的依赖标识。

```shell
npm init //初始化
```
如项目重复使用则使用npm install 即可。

![填写下关键字](http://img.blog.csdn.net/20150420204054517)

输入后回车后会出现一个确认的package.json预览如没有问题，请回车。

##Grunt的安装

这个比较简单
```shell
npm install grunt --save-dev 
```
//--save-dev表示在package.json中的devDependencies对象标识grunt及其版本

在安装基础库Grunt成功之后，再安装Grunt-cli
```shell
npm install grunt-cli --save-dev
```
安装完之后还要装[Grunt-init][3]用来创建gruntfile.js文件
```shell
npm install grunt-init --save-dev
```
之前我遇到的一个最大问题是权限的问题。待安装完之后，基本上grunt的环境基本搭建完成，下面我们来运行命令来创建[gruntfile.json][2]

```shell
grunt-init gruntfile
```
简单的回来几个问题i
```
Please answer the following:
[?] Is the DOM involved in ANY way? (Y/n) n
[?] Will files be concatenated or minified? (Y/n) y
[?] Will you have a package.json file? (Y/n) y
[?] Do you need to make any changes to the above before continuing? (y/N) n
```
之后gruntfile.js文件就创建完成了
```shell
drwxr-xr-x  6 root  staff   204B  4 21 10:32 node_modules
-rw-r--r--  1 root  staff   288B  4 21 10:54 package.json
```
这时可以看下gruntfile.js文件
```javascript
/*global module:false*/
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    // Metadata.
    pkg: grunt.file.readJSON('package.json'),
    banner: '/*! <%= pkg.title || pkg.name %> - v<%= pkg.version %> - ' +
      '<%= grunt.template.today("yyyy-mm-dd") %>\n' +
      '<%= pkg.homepage ? "* " + pkg.homepage + "\\n" : "" %>' +
      '* Copyright (c) <%= grunt.template.today("yyyy") %> <%= pkg.author.name %>;' +
      ' Licensed <%= _.pluck(pkg.licenses, "type").join(", ") %> */\n',
    // Task configuration.
    concat: {
      options: {
        banner: '<%= banner %>',
        stripBanners: true
      },
      dist: {
        src: ['lib/<%= pkg.name %>.js'],
        dest: 'dist/<%= pkg.name %>.js'
      }
    },
    uglify: {
      options: {
        banner: '<%= banner %>'
      },
      dist: {
        src: '<%= concat.dist.dest %>',
        dest: 'dist/<%= pkg.name %>.min.js'
      }
    },
    jshint: {
      options: {
        curly: true,
        eqeqeq: true,
        immed: true,
        latedef: true,
        newcap: true,
        noarg: true,
        sub: true,
        undef: true,
        unused: true,
        boss: true,
        eqnull: true,
        globals: {
          jQuery: true
        }
      },
      gruntfile: {
        src: 'Gruntfile.js'
      },
      lib_test: {
        src: ['lib/**/*.js', 'test/**/*.js']
      }
    },
    nodeunit: {
      files: ['test/**/*_test.js']
    },
    watch: {
      gruntfile: {
        files: '<%= jshint.gruntfile.src %>',
        tasks: ['jshint:gruntfile']
      },
      lib_test: {
        files: '<%= jshint.lib_test.src %>',
        tasks: ['jshint:lib_test', 'nodeunit']
      }
    }
  });

  // These plugins provide necessary tasks.
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-nodeunit');
  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-watch');

  // Default task.
  grunt.registerTask('default', ['jshint', 'nodeunit', 'concat', 'uglify']);

};
```
这里包括了文件的合并，压缩，node的单元测试，语法检测，文件检测和一些整体框架。

下面是一个Grunt的一个文件框架
```javascript
/*global module:false*/
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    //todo Task任务配置
  });

  // These plugins provide necessary tasks.
  grunt.loadNpmTasks('<%= Task任务 %>');
  //...
  // Task定制.
  grunt.registerTask('default', ['jshint', 'nodeunit', 'concat', 'uglify']);
};
```
下面就开始加载[css-sprite][4]模块
```shell
npm install css-sprite --save-dev
```
然后配置css-sprite任务
```javascript
css_sprite: {
            options: {
                'cssPath': '../images/images',
                'processor': 'scss',
                'orientation': 'vertical',
                'margin': 4
            },
            sprite: {
                options: {
                    'style': 'assets/sass/sprite.scss'
                },
                src: ['assets/sprites/*'],
                dest: 'build/images/sprite'
            },
            base64: {
                options: {
                    'base64': true
                },
                src: ['assets/sprites/*'],
                dest: 'assets/sass/base64.scss'
            }
        }
```

这里说明一下Grunt是分为任务，配置，目标这三个基本的概念，以css-sprite为例：

- **css_sprite:就是grunt的任务-Task**
- **options:就是grunt的配置-options，当然options也可以写道目标-target中**
- **sprite和base64分别是执行这个Task的不同的目标**

##实现过程

>目标：当需要进行png和jpg放进css-sprite目标文件夹的时候，就会生成生成sprite图与所对应的css或者sass预编译文件。
>实现原理：通过grunt-contrib-watch来检测目标文件夹的文件变化，如发生变化则执行css_sprite这个任务。

当task包都通过 npm install *** --save-dev进去之后，在Gruntfile.js配置如下：
```javascript
var LIVERELOAD_PORT = 35729;
var lrSnippet = require('connect-livereload')({ port: LIVERELOAD_PORT });
module.exports = function (grunt) {
    'use strict';
    // Project configuration.

    grunt.initConfig({
        // Metadata.
        pkg: grunt.file.readJSON('package.json'),
        meta: {
            version: '0.1.0'
        },
        banner: '/*! PROJECT_NAME - v<%= meta.version %> - ' +
            '<%= grunt.template.today("yyyy-mm-dd") %>\n' +
            '* http://PROJECT_WEBSITE/\n' +
            '* Copyright (c) <%= grunt.template.today("yyyy") %> ' +
            'YOUR_NAME; Licensed MIT */\n',
        // Task configuration.
        concat: {
            options: {
                banner: '<%= banner %>',
                stripBanners: true
            },
            js: {
                src: ['build/js/index.js'],
                dest: 'build/js/index.js'
            },
            css: {
                files: {
                    'build/css/base.css': 'build/css/base.css',
                    'build/css/index.css': 'build/css/index.css'
                }
            }
        },
        uglify: {
            options: {
                banner: '<%= banner %>'
            },
            js: {
                src: ['build/js/index.js'],
                dest: 'build/js/index.js'
            }
        },
        jshint: {
            options: {
                curly: true,
                eqeqeq: true,
                immed: true,
                latedef: true,
                newcap: true,
                noarg: true,
                sub: true,
                undef: true,
                unused: true,
                boss: true,
                eqnull: true,
                browser: true,
                globals: {
                    jQuery: true,
                    console: true,
                    module: false,
                    require: true
                }
            },
            gruntfile: {
                src: 'Gruntfile.js'
            },
            lib_test: {
                src: ['lib/**/*.js', 'test/**/*.js']
            }
        },
        qunit: {
            files: ['test/**/*.html']
        },
        watch: {
            gruntfile: {
                files: '<%= jshint.gruntfile.src %>',
                tasks: ['jshint:gruntfile']
            },
            lib_test: {
                files: '<%= jshint.lib_test.src %>',
                tasks: ['jshint:lib_test', 'qunit']
            },
            bower: {
                files: 'bower.json',
                tasks: ['wiredep']
            },
            html: {
                files: 'assets/**/*.tpl',
                tasks: ['html_template', 'wiredep']
            },
            sass: {
                files: 'assets/**/*.scss',
                tasks: ['sass:build', 'cssmin']
            },
            ts: {
                files: 'assets/**/*.ts',
                tasks: ['ts']
            },
            sprite: {
                files: 'assets/sprites/*',
                tasks: ['css_sprite']
            },
            livereload: {
                options: {
                    livereload: true
                    //check: true
                },
                files: ['build/css/*.css', 'build/js/*.js', 'html/*.html']
            }
        },
        html_template: {
            options: {
                beautify: {
                    indent_size: 2
                }
            },
            build_html: {
                expand: true,
                cwd: "assets/tpl",
                src: "*.tpl",
                dest: "./html"
            }
        },
        cssmin: {
            min: {
                files: [
                    {
                        expand: true,
                        cwd: 'build/css',
                        src: ['*.css', '!*.min.css'],
                        dest: 'build/css',
                        ext: '.min.css'
                    }
                ]
            }
        },
        sass: {
            build: {
                options: {
                    style: 'expanded'
                },
                files: {
                    'build/css/base.css': 'assets/sass/base.scss',
                    'build/css/index.css': 'assets/sass/index.scss'
                }
            }
        },
        css_sprite: {
            options: {
                'cssPath': '../images/images',
                'processor': 'scss',
                'orientation': 'vertical',
                'margin': 4
            },
            sprite: {
                options: {
                    'style': 'assets/sass/sprite.scss'
                },
                src: ['assets/sprites/*'],
                dest: 'build/images/sprite'
            },
            base64: {
                options: {
                    'base64': true
                },
                src: ['assets/sprites/*'],
                dest: 'assets/sass/base64.scss'
            }
        },
        ts: {
            build: {
                outDir: "build/js",
                src: ['assets/tsc/index.ts'],
                options: {
                    fast: 'never'
                }
            }

        },
        wiredep: {
            task: {
                src: [
                    'html/*.html',   // .html support...
                    'assets/sass/*.scss'  // .scss & .sass support...
                ]
            }
        },
        useminPrepare: {
            options: {
                dest: 'html/'
            },
            html: 'index.html'
        },
        usemin: {
            options: {
                assetsDirs: [
                    'build',
                    'build/images',
                    'build/css'
                ]
            },
            html: ['html/{,*/}*.html'],
            css: ['build/css/{,*/}*.css'],
            js: ['build/js/{,*/}*.js']
        },
        connect: {
            options: {
                port: 9000,
                open: true,
                //livereload: 35729,
                hostname: '0.0.0.0',
                base: "."
            },
            livereload: {
                options: {
                    middleware: function (connect, options) {
                        return [
                            lrSnippet,
                            // Serve static files.
                            connect.static(options.base.join("")),
                            // Make empty directories browsable.
                            connect.directory(options.base.join(""))
                        ];
                    }
                }
            }
        }
    });

    // These plugins provide necessary tasks.
    grunt.loadNpmTasks('grunt-contrib-concat');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-qunit');
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-contrib-cssmin');
    grunt.loadNpmTasks('grunt-contrib-connect');
    grunt.loadNpmTasks('grunt-contrib-sass');
    grunt.loadNpmTasks('grunt-html-template');
    grunt.loadNpmTasks('grunt-usemin');
    grunt.loadNpmTasks('grunt-ts');
    grunt.loadNpmTasks('grunt-wiredep');
    grunt.loadNpmTasks('css-sprite');
    // grunt.loadNpmTasks('grunt-filere');
    // Default task.
    grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
    grunt.registerTask('serve', function () {
        grunt.task.run(['connect', 'watch']);
    });
    grunt.registerTask('build', [
        'wiredep',
        'useminPrepare',
        'concat',
        'cssmin',
        'uglify',
        //'filerev',
        'usemin'
    ]);

};
```



---------

[1]: http://gruntjs.com/
[2]: http://gruntjs.com/getting-started
[3]:http://www.gruntjs.net/project-scaffolding
[4]:https://github.com/aslansky/css-sprite
