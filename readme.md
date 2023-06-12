# CockroachDB course

## About
Here is a course about [CockroachDB](https://www.cockroachlabs.com/) made with [slidev](https://sli.dev/).

## Start the server for presentation
```shell
npx slidev
```

### For a better view on a big screen
As explain [here](https://sli.dev/guide/faq.html#scale-the-canvas) you can scale the canvas.<br>
For that, uncomment this line in [slides.md](./slides.md) file. And change the value as desired.<br>
```shell
#canvasWidth: 950
```

## Exporting
### Export slides to pdf
```shell
npx slidev export
```

### Export slides to pdf with dark theme
```shell
npx slidev export --dark --output slides-export-dark.pdf
```

#### Help about slidev CLI
[Slidev Command Line Interface (CLI)](https://sli.dev/guide/install.html#command-line-interface-cli)

## Be up-to-date
Based on https://github.com/slidevjs/slidev/releases<br>
You can change the version number in [package.json](./package.json) file 
```json
    "@slidev/cli": "^0.40.14"
```

## TODO
 - [ ] English translation


## Author
Julien Bresson