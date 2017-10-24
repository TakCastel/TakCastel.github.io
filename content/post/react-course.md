+++
title = "Créer une application reactJS "
date = 2017-10-13
draft = false
tags = ["cours", "react"]
categories = ["formation"]

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++

Cet article est un témoin du cours React sur le courselab.

Ensemble, nous allons apprendre à développer un petit projet pour prendre en main la librairie React de Javascript, développée par Facebook. Ce projet vous permettra d'appréhender cette bibliothèque et en saisir tout son potentiel. 

Pour vous apprendre à bien maîtriser React, je pense que le mieux est de vous accompagner pas à pas dans la création d'un petit jeu de rôle par navigateur qui ne nécessitera pas de graphismes. Nous créerons un simple site à l'image des livres d'aventure que l'on appelle les livres-jeux.

Les livres dont vous êtes le héros sont des aventures narratives dans lesquelles vous incarnez un personnage et avez généralement entre 2 à 6 choix à la fin de chaque paragraphe. Ces choix vous emmènent vers d'autres paragraphes qui poursuivent l'aventure vers de nouveaux chemins. 

Si vous ne connaissez pas ce concept, je vous invite à lire l'article wikipédia sur les [défis fantastiques](https://fr.wikipedia.org/wiki/D%C3%A9fis_fantastiques), une collection très célèbre des années 80, grâce à laquelle j'ai eu l'idée de réaliser cette petite application web.

L'application démarre avec un fichier principal App.js :

    import React, { Component } from 'react'
    import { BrowserRouter as Router, Route } from 'react-router-dom'
    import { randomDice, saveCharacter, deleteCharacter } from './Services.js'
    import Header from './Header.js'
    import Game from './Game.js'
    import Character from './Character.js'
    import Home from './Home.js'
    import './App.css'

    class App extends Component {

    /* ===========================================================
    FONCTIONS DE CHARGEMENT DU PERSONNAGE
    =========================================================== */

    constructor(props) {
        super(props);
        this.state = {
        characterCreated: false, // Personnage créé ou pas
        name: '', // Nom du personnage
        tmpName: '', // Nom temporaire
        strenght: 0, // Force du personnage
        health: 0, // Santé du personnage
        chance: 0 // Chance du personnage
        }
    }

    // Chargement du personnage
    componentDidMount() {
        const name = localStorage.getItem('name')
        const strenght = localStorage.getItem('strenght')
        const health = localStorage.getItem('health')
        
        if (name !== null && strenght !== null && health !== null) {
        this.setState({
            characterCreated: true,
            name, 
            strenght: parseInt(strenght, 10), 
            health: parseInt(health, 10),
            chance: parseInt(health, 10)
        })
        } 
    }

    /* ===========================================================
    FONCTIONS DE CREATION DU PERSONNAGE
    =========================================================== */

    // Sauvegarde temporaire de l'entrée saisie dans l'input
    handleChange(event) {
        this.setState({tmpName :event.target.value})
    }

    // Transfert de la valeur temporaire à la valeur finale
    submitForm(event) {
        event.preventDefault()
        this.setState({name: this.state.tmpName})

        saveCharacter(this.state.tmpName, this.state.strenght, this.state.health, this.state.chance)
    }

    // Génération aléatoire des caractéristiques
    createCharacter(event) {
        event.preventDefault()
        const strenght = randomDice(7,12)
        const health = randomDice(14,24)
        const chance = randomDice(7,12)

        this.setState({
        characterCreated: true,
        strenght: strenght,
        health: health,
        chance: chance
        })

        saveCharacter(this.state.name, strenght, health, chance)
    }

    /* ===========================================================
    FONCTIONS DE SUPPRESSION DU PERSONNAGE
    =========================================================== */
    
    // Supprime les données du state et du local storage
    restartGame() {
        this.setState({
        characterCreated: false,
        name: '',
        tmpName: '',
        strenght: 0,
        health: 0,
        chance: 0
        })

        deleteCharacter()
    }

    /* ===========================================================
    AFFICHAGE DU COMPOSANT DANS LE DOM
    =========================================================== */

    render() {

        return (
        <Router>
            <div>
            <Header
                restartGame={() => this.restartGame()}
            ></Header>
            <Route 
                exact path="/" 
                render = {() => (
                <Home
                    characterCreated = {this.state.characterCreated}
                ></Home>
                )}
            ></Route>
            <Route 
                exact path="/game" 
                render = {() => (
                <Game
                    name = {this.state.name}
                    strenght = {this.state.strenght}
                    health = {this.state.health}
                    chance = {this.state.chance}
                ></Game>
                )}
            ></Route>
            <Route 
                exact path="/character" 
                render = {() => (
                <Character
                    characterCreated = {this.state.characterCreated}
                    name = {this.state.name}
                    tmpName = {this.state.tmpName}
                    strenght = {this.state.strenght}
                    health = {this.state.health}
                    chance = {this.state.chance}
                    handleChange = {(event) => this.handleChange(event)}
                    submitForm = {(event) => this.submitForm(event)}
                    createCharacter = {(event) => this.createCharacter(event)}
                ></Character>
                )}
            ></Route>
            </div>
        </Router>
        );
    }

    }

    export default App;

Puis un fichier Character.js :

    import React, { Component } from 'react'
    import { Link } from 'react-router-dom'

    class Character extends Component {

    constructor(props) {
        super(props);
        this.state = {
        submitted: false, // Indique si on a validé le nom
        };
    }

    // Renvoie un message d'erreur s'il n'y a pas de nom renseigné
    getWarning() {
        if (this.state.submitted && this.props.name === '') {
        return (
            <p> Vous devez renseigner un nom !</p>
        )
        }
    }

    // Affiche le formulaire de génération des caracs une fois le nom renseigné
    getCarac() {
        if (this.state.submitted && this.props.name !== '')
        return (
        <div>
            <p>Attribuer de nouvelles caractéristiques :</p>
            <button
            className="App-btn"
            onClick={(event) => this.props.createCharacter(event)}>
            Générer
            </button>
            <p>Force : {this.props.strenght}</p>
            <p>Santé : {this.props.health}</p>
            <p>Chance : {this.props.chance}</p>
        </div>
        )
    }

    // Affiche un lien vers le jeu une fois le personnage créé
    getPlay() {
        if (this.props.characterCreated === true && this.state.submitted === true) {
        return (
            <Link className="App-btn" to={'/game'}>Jouer</Link>
        )
        }
    }

    submitName(event) {
        this.props.submitForm(event)
        this.setState({submitted: true})
    }

    render() {
        return (
        <div className="App">
            <form
            className="App-form">
            <div className="App-form-group">
                <label htmlFor="name">Nom du personnage :</label>
                <input 
                htmlFor="name"
                id="name" 
                type="text" 
                value={this.props.tmpName}
                onChange={(event) => this.props.handleChange(event)}
                placeholder="John Doe"
                />
                <button
                className="App-btn"
                onClick={(event) => this.submitName(event)}>
                Nommer
                </button>
                {this.getWarning()}
                {this.getCarac()}
                {this.getPlay()}            
            </div>
            </form> 
        </div>
        );
    }

    }

    export default Character;

Et enfin 3 fichiers à part que sont le jeu dans Game.js :

    import React, { Component } from 'react'
    import Stats from './Stats.js'

    class Game extends Component {

    render () {
        return (
        <div>
            <Stats
            name = {this.props.name}
            strenght = {this.props.strenght}
            health = {this.props.health}
            chance = {this.props.chance}>
            </Stats>
        </div>
        )
    }

    }

    export default Game;

Les statistqiues du personnage dans Stats.js

    import React, { Component } from 'react'

    class Stats extends Component {

    render () {
        return (
        <ul>
            <li>Nom : {this.props.name}</li>
            <li>Force : {this.props.strenght}</li>
            <li>Santé : {this.props.health}</li>
            <li>Chance : {this.props.chance}</li>
        </ul>
        )
    }

    }

    export default Stats;

Et enfin des services externes (ici des fonctions utiles à la création aléatoire du personnage) dans Services.js :

    // Renvoie un entier aléatoire
    // entre une valeur min (incluse)
    // et une valeur max (incluse)
    export function randomDice(min, max) {
    min = Math.ceil(min)
    max = Math.floor(max)
    return Math.floor(Math.random() * (max - min +1)) + min
    }

    // Stock les données dans le navigateur
    export function saveCharacter(name, strenght, health, chance) {
    localStorage.setItem('name', name)
    localStorage.setItem('strenght', strenght)
    localStorage.setItem('health', health)
    localStorage.setItem('chance', chance)
    }

    // Enlève les données du navigateur
    export function deleteCharacter() { 
    localStorage.clear()
    }