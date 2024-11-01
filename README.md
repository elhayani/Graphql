#  Graphql
##  Angular
Pour utiliser GraphQL avec Angular, voici un guide étape par étape pour intégrer une API GraphQL dans une application Angular.
Prérequis
Assurez-vous que vous avez :
Une API GraphQL à laquelle se connecter.
Angular CLI et Node.js installés.
Étape 1 : Créer un projet Angular
Si vous n'avez pas encore de projet Angular, créez-en un en utilisant Angular CLI :
bash
Copier le code
ng new graphql-angular-app
cd graphql-angular-app

Étape 2 : Installer les dépendances nécessaires pour GraphQL
Angular peut se connecter à GraphQL via la bibliothèque Apollo Client, qui facilite les requêtes GraphQL et la gestion du cache côté client.
bash
Copier le code
npm install @apollo/client graphql

Étape 3 : Configurer Apollo Client dans Angular
Créez un module GraphQLModule pour configurer Apollo Client. Ce module va gérer la connexion à votre API GraphQL.
Dans le répertoire src/app, créez un fichier graphql.module.ts :
typescript
Copier le code
// src/app/graphql.module.ts
import { NgModule } from '@angular/core';
import { ApolloClientOptions, InMemoryCache } from '@apollo/client/core';
import { ApolloModule, APOLLO_OPTIONS } from 'apollo-angular';
import { HttpLink } from 'apollo-angular/http';

const uri = 'https://example.com/graphql'; // Remplacez avec l'URL de votre API GraphQL

export function createApollo(httpLink: HttpLink): ApolloClientOptions<any> {
  return {
    link: httpLink.create({ uri }),
    cache: new InMemoryCache(),
  };
}

@NgModule({
  exports: [ApolloModule],
  providers: [
    {
      provide: APOLLO_OPTIONS,
      useFactory: createApollo,
      deps: [HttpLink],
    },
  ],
})
export class GraphQLModule {}

Dans app.module.ts, importez ce nouveau module GraphQLModule et HttpClientModule :
typescript
Copier le code
// src/app/app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { GraphQLModule } from './graphql.module';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    HttpClientModule,
    GraphQLModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

Étape 4 : Définir une requête GraphQL
Supposons que nous ayons une API GraphQL qui retourne une liste d'utilisateurs avec leurs id et name. Créez un service UserService pour gérer les requêtes GraphQL.
Dans le dossier src/app, créez un fichier user.service.ts :
typescript
Copier le code
// src/app/user.service.ts
import { Injectable } from '@angular/core';
import { Apollo, gql } from 'apollo-angular';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
    }
  }
`;

@Injectable({
  providedIn: 'root',
})
export class UserService {
  constructor(private apollo: Apollo) {}

  getUsers(): Observable<any> {
    return this.apollo
      .watchQuery<any>({
        query: GET_USERS,
      })
      .valueChanges.pipe(map((result) => result.data.users));
  }
}

Étape 5 : Utiliser le service dans un composant
Dans le composant AppComponent, injectez le UserService pour récupérer la liste des utilisateurs et les afficher.
Mettez à jour app.component.ts :
typescript
Copier le code
// src/app/app.component.ts
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit {
  users: any[] = [];

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.userService.getUsers().subscribe((users) => {
      this.users = users;
    });
  }
}

Ensuite, dans app.component.html, affichez les utilisateurs récupérés :
html
Copier le code
<!-- src/app/app.component.html -->
<div *ngIf="users.length > 0">
  <h2>Liste des utilisateurs :</h2>
  <ul>
    <li *ngFor="let user of users">
      {{ user.name }} (ID: {{ user.id }})
    </li>
  </ul>
</div>
<div *ngIf="users.length === 0">
  <p>Aucun utilisateur trouvé.</p>
</div>

Étape 6 : Lancer l'application
Démarrez votre application Angular pour vérifier que tout fonctionne :
bash
Copier le code
ng serve

Accédez à l'application via http://localhost:4200, et vous devriez voir la liste des utilisateurs retournée par l'API GraphQL.
Résumé
Apollo Client : Configurez Apollo Client pour se connecter à votre API GraphQL.
Définir des requêtes : Utilisez le gql d’Apollo pour définir vos requêtes et mutations GraphQL.
Services et Composants : Injectez votre UserService dans les composants pour exécuter les requêtes.
Avec cette configuration de base, vous pouvez étendre votre intégration GraphQL en ajoutant des mutations, en gérant le cache Apollo, et en intégrant d'autres fonctionnalités comme la pagination ou les souscriptions pour les données en temps réel.

##  Spring Boot

Pour intégrer une API GraphQL avec Spring Boot, voici les étapes essentielles pour configurer GraphQL côté backend avec Spring Boot et exposer des requêtes et mutations que l’application Angular pourra interroger.
Prérequis
Assurez-vous d’avoir :
Un projet Spring Boot (idéalement, vous pouvez en créer un sur Spring Initializr).
Étape 1 : Configurer les dépendances GraphQL dans Spring Boot
Ajoutez les dépendances nécessaires à votre projet. Utilisez Maven ou Gradle, selon votre configuration. Voici l'exemple pour Maven :
xml
Copier le code
<dependencies>
    <dependency>
        <groupId>com.graphql-java-kickstart</groupId>
        <artifactId>graphql-spring-boot-starter</artifactId>
        <version>11.1.0</version>
    </dependency>
    <dependency>
        <groupId>com.graphql-java-kickstart</groupId>
        <artifactId>graphiql-spring-boot-starter</artifactId>
        <version>11.1.0</version>
    </dependency>
    <dependency>
        <groupId>com.graphql-java-kickstart</groupId>
        <artifactId>graphql-java-tools</artifactId>
        <version>11.1.0</version>
    </dependency>
</dependencies>

Ces dépendances vous permettent de configurer facilement un serveur GraphQL avec Spring Boot et d’utiliser GraphiQL, une interface web pour tester vos requêtes GraphQL.
Étape 2 : Créer un schéma GraphQL
Définissez un fichier de schéma GraphQL (par exemple schema.graphqls) dans le dossier src/main/resources/graphql/. Ce fichier contient les définitions de requêtes (Query), mutations (Mutation), et les types.
Par exemple, supposons que nous ayons un type User :
graphql
Copier le code
# src/main/resources/graphql/schema.graphqls

type User {
    id: ID!
    name: String!
    email: String!
}

type Query {
    getUserById(id: ID!): User
    getAllUsers: [User]
}

type Mutation {
    createUser(name: String!, email: String!): User
    deleteUser(id: ID!): Boolean
}

Étape 3 : Créer l'entité User
Dans src/main/java/com/example/demo/entity, créez une classe User qui représente le modèle de données.
java
Copier le code
// src/main/java/com/example/demo/entity/User.java
package com.example.demo.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;

    // Constructors, getters and setters
    public User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
    // ...
}

Étape 4 : Créer le Repository UserRepository
Créez un repository UserRepository pour gérer les opérations CRUD avec la base de données.
java
Copier le code
// src/main/java/com/example/demo/repository/UserRepository.java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}

Étape 5 : Créer les resolvers GraphQL pour les requêtes et mutations
Dans src/main/java/com/example/demo/resolver, créez les classes de résolveur pour les requêtes (QueryResolver) et les mutations (MutationResolver).
Query Resolver
java
Copier le code
// src/main/java/com/example/demo/resolver/UserQueryResolver.java
package com.example.demo.resolver;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import com.coxautodev.graphql.tools.GraphQLQueryResolver;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class UserQueryResolver implements GraphQLQueryResolver {

    private final UserRepository userRepository;

    public UserQueryResolver(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}

Mutation Resolver
java
Copier le code
// src/main/java/com/example/demo/resolver/UserMutationResolver.java
package com.example.demo.resolver;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import com.coxautodev.graphql.tools.GraphQLMutationResolver;
import org.springframework.stereotype.Component;

@Component
public class UserMutationResolver implements GraphQLMutationResolver {

    private final UserRepository userRepository;

    public UserMutationResolver(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User createUser(String name, String email) {
        User user = new User(name, email);
        return userRepository.save(user);
    }

    public Boolean deleteUser(Long id) {
        userRepository.deleteById(id);
        return true;
    }
}

Étape 6 : Lancer l'application et tester avec GraphiQL
Lancez votre application Spring Boot :
bash
Copier le code
./mvnw spring-boot:run

Ensuite, ouvrez GraphiQL à l'adresse http://localhost:8080/graphiql pour tester vos requêtes et mutations.
Exemple de requête GraphQL
Pour obtenir un utilisateur par ID :
graphql
Copier le code
query {
    getUserById(id: 1) {
        id
        name
        email
    }
}

Exemple de mutation GraphQL
Pour créer un nouvel utilisateur :
graphql
Copier le code
mutation {
    createUser(name: "Alice", email: "alice@example.com") {
        id
        name
        email
    }
}

Résumé
Configurer les dépendances GraphQL : Utilisez les bibliothèques graphql-spring-boot-starter et graphql-java-tools.
Définir le schéma GraphQL : Déclarez vos types, requêtes et mutations.
Créer les résolveurs : Implementez GraphQLQueryResolver et GraphQLMutationResolver.
Tester avec GraphiQL : Utilisez l'interface GraphiQL pour vérifier les requêtes et mutations.
Cela vous permet de construire une API GraphQL dans Spring Boot, prête à être consommée par votre application Angular.
4o
Vous avez dit :
mac terminal avec plusieur onglets
ChatGPT a dit :
ChatGPT
Pour ouvrir plusieurs onglets dans le Terminal sur Mac, voici plusieurs méthodes que vous pouvez utiliser :
1. Utiliser les raccourcis clavier
Ouvrir un nouvel onglet : Command + T
Naviguer entre les onglets : Command + Shift + [ (pour aller à l'onglet précédent) et Command + Shift + ] (pour aller à l'onglet suivant)
Fermer un onglet : Command + W
2. Ouvrir plusieurs onglets avec des commandes spécifiques
Pour lancer plusieurs onglets avec des commandes spécifiques, vous pouvez créer un script shell. Par exemple :
bash
Copier le code
#!/bin/bash
# Ouvrir un nouvel onglet et exécuter une commande dans chaque onglet

osascript <<EOF
tell application "Terminal"
    do script "echo 'Premier onglet'; ls"  -- ouvre le premier onglet avec une commande
    delay 1
    do script "echo 'Deuxième onglet'; pwd" -- ouvre un second onglet avec une autre commande
end tell
EOF

Pour exécuter ce script :
Enregistrez-le dans un fichier (par exemple, open_tabs.sh).
Donnez-lui les permissions d'exécution : chmod +x open_tabs.sh.
Exécutez le script : ./open_tabs.sh.
3. Utiliser iTerm2 (une alternative avec plus de fonctionnalités)
iTerm2 est une alternative au Terminal par défaut de macOS qui offre des fonctionnalités avancées :
Onglets multiples
Fenêtres divisées pour exécuter plusieurs sessions côte à côte
Raccourcis personnalisés et bien plus.
Après l'installation, vous pouvez ouvrir de nouveaux onglets avec Command + T et même diviser les fenêtres en panneaux (avec Command + D pour une division horizontale et Command + Shift + D pour une division verticale).
Ces méthodes vous permettront de mieux gérer et organiser plusieurs sessions de terminal.

