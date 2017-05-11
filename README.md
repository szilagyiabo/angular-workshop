# Angular Workshop

This is a step-by-step guide to start learning Angular2

## Prerequisites

Node.js and npm are essential to this project. 
I will use **node v4.9.5** and **npm 3.10.10**.

You can install them from official Node.js website: https://nodejs.org/en/
(LTS is recommended)

After *node* and *npm* are installed, download the latest *ng-cli* (official command line interface for Angular2)
`npm install -g @angular/cli`
https://github.com/angular/angular-cli

## Lesson 1
Now we can create our first Angular2 project. It is as simple as it needs to be:
```bash
cd ~/Work
ng new weather-app
```

This command will generate all files we need. Check out what the folder contains!

If we now navigate to the created folder, we can start running our application:
```bash
cd weather-app
ng serve
```

This command will build our app, and start a new "development server", where we can check what we done. Check out `http://localhost:4200` !

## Lesson 2
### Let's modify something

Now we have our basic project up and ready. Let's do some experiment!
I usually use Visual Studio Code for code editing (or vim), but any text editor or IDE could be ok.

Let's open *src/app/app.component.html* file.
```html
<h1>
  {{title}}
</h1>
```
Not so sophisticated, right? :)
Let's add some paragraph to it:
```html
<h1>
  {{title}}
</h1>
<p>
  This is awesome!
</p>
```
And check out our browser!
This is awesome, right? :)
---
Let's do more experiment!
Open up *src/app/app.component.css* file. It empty now. Make a new class, called *awesome-text*
```css
.awesome-text {
    color: #23ca12;
}
```

And modify *app.component.html* file, to use this class:
```html
<h1>
  {{title}}
</h1>
<p class="awesome-text">
  This is awesome!
</p>
```
Check your browser! It's working! :)

## Lesson 3
### Adding new component
What is a component? And how this all works?
This is the sshematic architecture of an Angular2 app:
![Angular2 architecture](https://angular.io/resources/images/devguide/architecture/overview2.png)
https://angular.io/docs/ts/latest/guide/architecture.html

Does it looks complicated? Yes.
But trust me, it is not so.

When we create a new component, basically the center of the image will be created. The HTML and CSS files will go into the "Template" box. And the .ts file will be the "Component" box with the gears.

The other part of image will be discussed later.

To add a new component the one and only thing you have to do is:
`ng g component temperature-box`
This will generate a new folder, with 4 new files, and modify another. Look like this:
```bash
installing component
  create src/app/temperature-box/temperature-box.component.css
  create src/app/temperature-box/temperature-box.component.html
  create src/app/temperature-box/temperature-box.component.spec.ts
  create src/app/temperature-box/temperature-box.component.ts
  update src/app/app.module.ts
```

Modify the newly generated *temperature-box.component.html* file:
```html
<p>
  It 32&deg; out there.
</p>
```
This will tell us, how is the weater today. It's a little dummy so far, but will be smarter soon. :)
Now add this *weater-box* to our application! Open up the *app.component.html* file, and add the following line to the end of the file:
```html 
<app-temperature-box></app-temperature-box> 
```
Check out the browser! It's a little hot out there.
But what the hack is this tag? Ain't nobody get tag for this!
The short answer is: We did it.

Check out the *temperature-box.component.ts*! We defined this component with the selector *'app-temperature-box'*. And any time we use this selector, this will inject our template to the code. (the template which given with the *templateUrl*, and *styleUrls*)


## Lesson 4
### Adding a new service
**What a service is?**
A service contains the business logic of our app. Do API calls, store states, calculate complex things, etc.

Creating a new service is simple as creating a component:
`ng g service open-weather-api`

This command will generate us two files:
```bash
installing service
  create src/app/open-weather-api.service.spec.ts
  create src/app/open-weather-api.service.ts
  WARNING Service is generated but not provided, it must be provided to be used
```
This warning message is important.
We have to manually add it to our *app.module.ts* file, so let's do it!
Add this line after the imports:
`import { OpenWeatherApiService } from './open-weather-api.service';`
This will import our new service.
After this, we have to add this service to the providers:
```js
...
providers: [
    OpenWeatherApiService
],
...
```

Now we can use our component in every component of this module.
So let's do our first service function! Open up the *open-weather-api.service.ts* file, and add a new funtion after the constructor of the service, like this.
```ts
import { Injectable } from '@angular/core';

@Injectable()
export class OpenWeatherApiService {

  constructor() { }

  getCurrentWeather() {
    return 52;
  }
}
```
It's dummy as hell, but will be smarter soon. :)
Now we can use this service via a simple *dependency injection*. To do so, open *temperature-box.component.ts*, and add a new import:
```ts
import { OpenWeatherApiService } from '../open-weather-api.service';
```
Than modify the constructor like this:
```ts
constructor(private openWeatherApiService: OpenWeatherApiService) { }
```
Thats it. We have just dependency inject our service. Cool, right? :)
Now we can acceess the service vie the *openWeatherApiService* variable.
Let's create a new variable in this file which will store the current temperature from the service. This variable will be displayed in the HTML file too. So the *temperature-box.component.ts* file will be look like:
```ts
import { OpenWeatherApiService } from '../open-weather-api.service';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-temperature-box',
  templateUrl: './temperature-box.component.html',
  styleUrls: ['./temperature-box.component.css']
})
export class TemperatureBoxComponent implements OnInit {

  currentTemp: Number;

  constructor(private openWeatherApiService: OpenWeatherApiService) { }

  ngOnInit() {
    this.currentTemp = this.openWeatherApiService.getCurrentWeather();
  }
}
```

And the *temperature-box.component.html* file will be look like this:
```html
<p>
  It {{currentTemp}}&deg; out there.
</p>
```
(We could display variables in template by useing double curly brackets.)
Check out the browser! It's getting hotter and hotter.

## Lesson 5
### Calling API
It's time to get real data. 
**Where do we need to call the API?**
Service.
Fortunately, we have one.
So let's do it! Open *open-weather-api.service.ts* file, and modify like this:
```ts
import { Injectable } from '@angular/core';

import { Http } from '@angular/http';
import { Observable } from 'rxjs/Observable';

import 'rxjs/Rx';

@Injectable()
export class OpenWeatherApiService {

  apiEndpoint = 'http://api.openweathermap.org/data/2.5/weather?q=Budapest,hu&mode=json&APPID={{APPID}}';

  constructor(private http: Http) { }

  getCurrentWeather(): Observable<Object> {
    return this.http.get(this.apiEndpoint)
      .map(response => {
        const weatherBody = JSON.parse(response['_body']);
        return weatherBody || { };
      });
  }
}
```
What we modified here are some import to use http requests, and Observable to make it observable :)
We pimped up out *getCurrentWeather()* function to call OpenWeatherMap API, and parse the response body. 
Cool our service is now do what she do best.

Now we have to update our *weather-box.component.ts* too, to handle this service call.
If we want to use an Observer, we have to subscribe on it. It's not too complicated, just call its *subscribe()* function. So our component will be look like this:
```ts
import { OpenWeatherApiService } from '../open-weather-api.service';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-temperature-box',
  templateUrl: './temperature-box.component.html',
  styleUrls: ['./temperature-box.component.css']
})
export class TemperatureBoxComponent implements OnInit {

  currentTemp: Number;

  constructor(private openWeatherApiService: OpenWeatherApiService) { }

  ngOnInit() {
    this.getCurrentWeatherData();
  }

  getCurrentWeatherData() {
    this.openWeatherApiService.getCurrentWeather().subscribe(response => {
      this.currentTemp = +response['main']['temp'] - 273.15;
    });
  }
}
```
Here we just created a new function *getCurrentWeatherData()*, which handle response from service. In this case, we store the temperature data to our *currentTemp* variable. (with K -> C convert)
And that's all. Refresh your browser and see some magic. :)

To make it near-realtime, and do auto refresh, we need to put the function call in a *setInterval()* function, like so:
```ts
ngOnInit() {
    setInterval(() => this.getCurrentWeatherData(), 60000);
}
```
This will refresh the current weather in every minutes.

## Lesson 6
### Deployment
Angular CLI provides us an awesome tool to build our app.
`ng build --prod --aot`
This command will generate a *dist* folder, where the compiled, minified, everyfied files are located. To delpoy the app we only need this folder. This way we can easily integrate this into any CI/CD tool. 
OR
The app could be easily hosted on AWS S3. There is CLI for aws, and also for s3, so its highly scriptable. I recommend to read my article about this topic: https://medium.com/@szilagyiabo/angular2-s3-love-deploy-to-cloud-in-6-steps-3f312647a659


