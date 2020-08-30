# ParadigmasDeProgramacion

import donal.*
import wollok.game.*
import extras.*

object fin inherits Visual (position = new Position(x = 8, y = 1), image = 'fin.png') {

//	override method image() {
//		if (donal.dinero()>999) return 'win.png'
//		else return 'fin.png'
//	}
	 
	method continuar() {
		game.onTick(700, "GRAVEDAD", { donal.caer(1)})
		donal.vidas(3)
		donal.dinero(0)
		donal.position(game.at(10, 10))

		if (!game.hasVisual(torreTrump)) {
			torreTrump.aparecer()
		}
		if (!game.hasVisual(bolsonaro)) {
			bolsonaro.aparecer()
		}
		if (!game.hasVisual(britanico)) {
			britanico.aparecer()
		}
		dolar.position(game.at(1, 1))
		coronavirus.position(game.at(9, 9))
		doctor.position(game.at(7, 7))
		game.removeVisual(self)
	}
	
	method finDelJuego(){
		game.removeTickEvent("GRAVEDAD")
		game.addVisual(self)
	}
	
	method terminar() {
		game.removeVisual(self)
		game.schedule(2000, { game.stop()})
	}

}

object vida inherits Visual (position = new Position(x = 22, y = 12)){
	
	override method image() = "corazon" + donal.vidas().toString() + ".png"
	
}

object unidad inherits Visual (position = new Position(x = 24, y = 11)){
	
	override method image() = donal.dinero() - ((donal.dinero()/10).truncate(0)*10).toString() + ".png"
	
}

object decena inherits Visual (position = new Position(x = 23, y = 11)){

	override method image() = if (donal.dinero()>=100)
						{(((donal.dinero() - centena.c()*100)/10).truncate(0)).toString() + ".png"}
					else {(donal.dinero()/10).truncate(0).toString() + ".png"}
}

object centena inherits Visual (position = new Position(x = 22, y = 11)){

	override method image() = self.c().toString() + ".png"

	method c() = (donal.dinero() / 100).truncate(0)
}

object signoPeso inherits Visual (position = new Position(x = 21, y = 11), image="dolar.png"){}

----

import wollok.game.*
import interface.*
import extras.*

object donal inherits Visual (position = new Position(x = 10, y = 10), image = "donalsito.png") {

	var property dinero = 0
	var property vidas = 3
	var property estatico = false

	method move(nuevaPosicion) {
		self.position(nuevaPosicion)
	}

	method colisionarCon(unPersonaje) {
		dinero = dinero + unPersonaje.dineroQueLeOtorga()
		dolar.mover()
	}

	method caer(altura) {
		if (!estatico) {
			self.position(new Position(x = self.position().x().limitBetween(1.5, 23), y = (self.position().y() - altura).limitBetween(0, 11)))
		}
	}	
}


----

import wollok.game.*
import interface.*
import donal.*

class Visual {

	var property position
	var property image

	method teEncontro() {
	}
	
}

class DanDinero inherits Visual {

	var property dineroQueLeOtorga =  12

	override method teEncontro() {
		self.darDinero()
	}

	method aparecer() {			
		game.addVisual(self)
	}

	method desaparecer() {
		game.removeVisual(self)
	}
	
	method darDinero() { 
		donal.dinero(donal.dinero() + dineroQueLeOtorga)
		if (donal.dinero() > 999) {fin.finDelJuego()}
		self.desaparecer()	
}

}

object torreTrump inherits DanDinero (position = new Position(x = 4, y = 6), image = "torre_trump.png") {

}

object bolsonaro inherits DanDinero (position = new Position(x = 8, y = 4), image = "bolsonaro.png") {

}

object britanico inherits DanDinero (position = new Position(x = 11, y = 8), image = "britanico.png") {

}

class QuitanDinero inherits Visual {

	var property dineroQueLeQuita = 9

	override method teEncontro() {
		self.quitarDinero()
	}
	
	method quitarDinero() { 
		donal.dinero(donal.dinero() - self.dineroQueLeQuita()).max(0)
		game.say(self, "Has perdido dolares")
	}
	
}

object coreano inherits QuitanDinero (position = new Position(x = 19, y = 6), image = "coreano.png") {

}

object chino inherits QuitanDinero (position = new Position(x = 19, y = 8), image = "chino2.png") {

}

class Mortal inherits Visual {

	override method teEncontro() {
		game.removeTickEvent("GRAVEDAD")
		game.addVisual(fin)
	}

}

object jon inherits Mortal (image = "jon.png") {

	override method position() = new Position(x = donal.position().x().min(25), y = 0)
}

object bomba inherits Mortal (image = "bombaDer.png") {

	override method position() = new Position(x = donal.position().x().min(24), y = 12)
}

class Paralizador inherits Visual {

	const tiempo

	override method teEncontro() {
		donal.estatico(true)
		game.schedule(tiempo, { donal.estatico(false)})
		game.say(self, "Entraste en cuarentena")
	}

}

object muro inherits Paralizador (position = new Position(x = 5, y = 5), image = "muroEEUUyMEX.png", tiempo = 2000) {

}

object alberto inherits Paralizador (position = new Position(x = 8, y = 2), image = "alberto.png", tiempo = 3000) {

}

object dolar inherits DanDinero (position = new Position(x = 1, y = 1), image = "dolarr.png") {

	method mover() {
		const x = 1.randomUpTo(game.width().min(24)).truncate(0)
		const y = 1.randomUpTo(game.height().min(12)).truncate(0)
		position = game.at(x, y)
	}

	override method teEncontro() {
		donal.colisionarCon(self)
	}

}

class QuitanVida inherits Visual {

	method vidaQueleSaca() = 1

	override method teEncontro() {
		self.quitarVida()
	}
	
	method quitarVida() {
		donal.vidas((donal.vidas() - self.vidaQueleSaca()).min(0)) 
			
		if (donal.vidas() == 0) {
			fin.finDelJuego()}
		
		self.accion()			
		}		
		
		method accion(){game.say(self,"SUERTE PARA LA PROXIMA" )}
	
}

object putin inherits QuitanVida (position = new Position(x = 15, y = 8), image = "putin.png"){
	
}

object africanosBailarines inherits QuitanVida (position = new Position(x = 13, y = 5), image = "africanosQueBailan1.png") {

}

object coronavirus inherits QuitanVida (position = new Position(x = 9, y = 9), image = "coronavirus.png") {

	method mover() {
		const x = 1.randomUpTo(game.width()).truncate(0)
		const y = 1.randomUpTo(game.height() - 1).truncate(0)
		position = game.at(x, y)
	}
	
	override method accion(){
		game.say(self,"PERDISTE UNA VIDA, CUIDADO")
		self.mover()
	}

}

object doctor inherits Visual (position = new Position(x = 7, y = 7), image = "doctor.png") {

	method mover() {
		const x = 1.randomUpTo(game.width()).truncate(2)
		const y = 1.randomUpTo(game.height() - 1).truncate(2)
		position = game.at(x, y)
	}

	method darVida() {
		if (donal.vidas() < 3) {
			donal.vidas(donal.vidas() + 0.5)
			game.say(self, "ganaste vida")
		} else {
			game.say(self, "Tenes más vida que Mirtha")
		}
		self.mover()
	}

	override method teEncontro() {
		self.darVida()
	}

}

object angela inherits Visual (position = new Position(x = 12, y = 7), image = "angelaMerkel.png") {

	override method teEncontro() {
		donal.dinero(0)
	}

}


 
