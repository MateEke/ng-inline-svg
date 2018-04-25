# ng-inline-svg

[![NPM](https://nodei.co/npm/ng-inline-svg.png?compact=true)](https://nodei.co/npm/ng-inline-svg)

**[Demo](https://echeung.me/ng-inline-svg)**

*Formerly called [ng2-inline-svg](https://github.com/arkon/ng2-inline-svg)*

Angular directive for inserting an SVG inline within an element, allowing for easily styling
with CSS like `fill: currentColor;`.

This is based on [md-icon](https://github.com/angular/material2/tree/master/src/lib/icon),
except this is meant purely for inserting SVG files within an element, without the extra things like
font icons.


## Installation

```shell
npm install --save ng-inline-svg
```


## Usage

Add `HttpClientModule` and `InlineSVGModule` to your list of module imports:

```typescript
import { HttpClientModule } from '@angular/common/http';
import { InlineSVGModule } from 'ng-inline-svg';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule, InlineSVGModule],
  bootstrap: [AppComponent]
})
class AppModule {}
```

The base URL will be automatically fetched based on your `<base>` tag. If you want to configure the base URL manually, you can use the `forRoot` function:

```typescript
InlineSVGModule.forRoot({ baseUrl: 'https://mysite.com/myapp/' })
```

If you want to configure the base URL later, use the service's function:

```typescript
import { SVGCacheService } from 'ng-inline-svg';

class YourComponent {
  constructor(svgService: SVGCacheService) {
    svgService.setBaseUrl({ baseUrl: 'https://mysite.com/myapp/' });
  }
}
```

You can then use the directive in your templates:

```typescript
@Component({
  selector: 'app',
  template: `
    <div class="my-icon" aria-label="My icon" [inlineSVG]="'/img/image.svg'"></div>
  `
})
export class AppComponent {
}
```

The SVG file (if found) will be inserted *inside* the element with the `[inlineSVG]` directive. Support for icons using the [`symbol` method](https://css-tricks.com/svg-symbol-good-choice-icons/) is also supported (e.g. `[inlineSVG]="'#shape-id'"`).

### Options

#### Inputs

| Property name | Type | Default | Description |
| ------------- | ---- | ------- | ----------- |
| cacheSVG | boolean | `true` | Caches the SVG based on the absolute URL. Cache only persists for the (sessional) lifetime of the page. |
| prepend | boolean | `false` | Inserts before the first child instead of appending, overwrites `replaceContents` |
| replaceContents | boolean | `true` | Replaces the contents of the element with the SVG instead of just appending it to its children. |
| injectComponent | boolean | `false` | Injects an `<inline-svg>` component containing the SVG inside the element with the directive. |
| removeSVGAttributes | string[] | | An array of attribute names to remove from all child elements within the SVG. |
| forceEvalStyles | boolean | `false` | Forces embeded style tags' contents to be evaluated (for IE 11). |
| evalScripts | `'always'`, `'once'`, `'none'` | `'always'` | Whether to evaluate embedded scripts in the loaded SVG files. |
| fallbackImgUrl | string | | URL for a regular image to be displayed as a fallback if the SVG fails to load. |
| onSVGLoaded | `(svg: SVGElement, parent: Element \| null) => SVGElement` | | Lifecycle hook that allows the loaded SVG to be manipulated prior to insertion. |

#### Outputs

| Property name | Callback arguments | Description |
| ------------- | ------------------ | ----------- |
| onSVGInserted | `e: SVGElement` | Emits the `SVGElement` post-insertion. |
| onSVGFailed | `e: any` | Emits when there is some error (e.g. embed SVG not supported, fetch failed, etc.) |

### Server-side rendering with Angular Universal

The SVG files can also be rendered server-side. For this to work, you have to set the `baseURL`, since Angular needs to have an absolute URL to retrieve the files server-side and we're not able to get your baseURL automatically in a server-side environment. See manual for setting `baseURL` above.

Here is one way to achieve this dynamically by adding an app initalizing service which sets the base URL based on the environment it runs.

*app.module.ts*:
```typescript
import { NgModule, APP_INITIALIZER } from '@angular/core';
 
import { AppLoadService } from './app-load.service';
 
export function initApp(appLoadService: AppLoadService) {
    return () => appLoadService.initializeApp();
}
 
@NgModule({
  providers: [
    AppLoadService,
    { provide: APP_INITIALIZER, useFactory: initApp, deps: [AppLoadService], multi: true },
  ]
})
export class AppLoadModule { }
```

*app-load.service.ts*:
```typescript
import { Injectable } from '@angular/core';
 
import { APP_SETTINGS } from '../settings';
 
@Injectable()
export class AppLoadService {
  constructor() { }
 
  initializeApp() {
    // Do what ever conditions you need to set this, e.g. checking for server-side rendering
    // and only set baseURL when server-side rendered if you want.
    // Example: When the server-side rendered app runs on localhost:3000, make sure baseURL is
    // http://localhost:3000 and make sure the express server is configured properly to 
    // allow the URL of the asset folders storing the svg files.
    // ...

    // Set baseURL
    this.svgService.setBaseUrl({ baseUrl: APP_SETTINGS.myBaseURLOfTheCurrentEnvironment });
  }
}
```
