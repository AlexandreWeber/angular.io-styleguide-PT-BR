### Responsábilidade Única

Aplique o conceito de responsabilidade única [SRP][https://en.wikipedia.org/wiki/Single_responsibility_principle] para todos os componentes, serviços, pipes, diretivas e outros arquivos do projeto. Isso ajuda a manter a aplicação mais limpa, fácil de ler e manutenir, e mais fácil para testar.

**Faça:** Defina apenas uma coisa, como um serviço ou componente, por arquivo.

**Importante:** Considere limitar o arquivo a 400 linhas de código.

**Por que?** Um componente por arquivo é muito mais fácil para ler, manter e evitar problemas na comparação de fontes quando utilizado um controlador de versão (git).

**Por que?** Um componente por aquivo evita "bugs escondidos" que podem ocorrer com a combinação de um ou mais componentes em um mesmo arquivo onde eles podem compartilhar variáveis, criar closures inesperadas ou acoplamentos indesejados com dependências.

**Por que?** Um componente único por aquivo pode ser a exportação padrão para aquele arquivo, o que facilita o lazy loading.

O ponto principal é tornar código reutilizave, fácil de ler, e menos propenso a erros.

O exemplo negativo abaixo define o AppComponent, bootstraps o aplicativo, define o modelo *Hero* e realiza uma requisição no servidor, tudo isso no mesmo arquivo. *Não faça isso.*

**app/heroes/hero.component.ts**
```
/* avoid */
import { Component, NgModule, OnInit } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

class Hero {
  id: number;
  name: string;
}

@Component({
  selector: 'app-root',
  template: `
      <h1>{{title}}</h1>
      <pre>{{heroes | json}}</pre>
    `,
  styleUrls: ['app/app.component.css']
})
class AppComponent implements OnInit {
  title = 'Tour of Heroes';

  heroes: Hero[] = [];

  ngOnInit() {
    getHeroes().then(heroes => (this.heroes = heroes));
  }
}

@NgModule({
  imports: [BrowserModule],
  declarations: [AppComponent],
  exports: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}

platformBrowserDynamic().bootstrapModule(AppModule);

const HEROES: Hero[] = [
  { id: 1, name: 'Bombasto' },
  { id: 2, name: 'Tornado' },
  { id: 3, name: 'Magneta' }
];

function getHeroes(): Promise<Hero[]> {
  return Promise.resolve(HEROES); // TODO: get hero data from the server;
}
```
É uma melhor prática distribuir o componente e suas classes suportados em arquivos separados, cada um com a sua responsabilidade.

**main.ts**
```
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```
**app/app.module.ts**
```
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { RouterModule } from '@angular/router';

import { AppComponent } from './app.component';
import { HeroesComponent } from './heroes/heroes.component';

@NgModule({
  imports: [
    BrowserModule,
  ],
  declarations: [
    AppComponent,
    HeroesComponent
  ],
  exports: [ AppComponent ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```
**app/app.component.ts**
```
import { Component } from '@angular/core';

import { HeroService } from './heroes';

@Component({
  selector: 'toh-app',
  template: `
      <toh-heroes></toh-heroes>
    `,
  styleUrls: ['./app.component.css'],
  providers: [HeroService]
})
export class AppComponent {}
```
**app/heroes/heroes.component.ts**
```
import { Component, OnInit } from '@angular/core';

import { Hero, HeroService } from './shared';

@Component({
  selector: 'toh-heroes',
  template: `
      <pre>{{heroes | json}}</pre>
    `
})
export class HeroesComponent implements OnInit {
  heroes: Hero[] = [];

  constructor(private heroService: HeroService) {}

  ngOnInit() {
    this.heroService.getHeroes()
      .then(heroes => this.heroes = heroes);
  }
}
```
**app/heroes/shared/hero.service.ts**
```
import { Injectable } from '@angular/core';

import { HEROES } from './mock-heroes';

@Injectable()
export class HeroService {
  getHeroes() {
    return Promise.resolve(HEROES);
  }
}
```
**app/heroes/shared/hero.model.ts**
```
export class Hero {
  id: number;
  name: string;
}
```
**app/heroes/shared/mock-heroes.ts**
```
import { Hero } from './hero.model';

export const HEROES: Hero[] = [
  { id: 1, name: 'Bombasto' },
  { id: 2, name: 'Tornado' },
  { id: 3, name: 'Magneta' }
];
```
Na medida que a aplicação cresce, essa regra se torna ainda mais importante.




