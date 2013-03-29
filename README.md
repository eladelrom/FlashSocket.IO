# FlashSocket.IO

Flash library to facilitate communication between Flex applications and Socket.IO servers.

The actual websocket communication is taken care of by my fork of gimite/web-socket-js project.

This project wraps that and facilitates the hearbeat and en/decoding of messages so they work with Socket.IO servers

# Checkout

Because this project makes use of git submodules you must make use of the recursive clone.

git clone --recursive git://github.com/simb/FlashSocket.IO.git

# Building

Because this library is dependent on the websocket library, you must add the support/websocket-js path to you source path in flash builder or your build files.

# Usage

For Socket.io 0.7 and 0.8 or 0.9 you will need to use the Beta swc that you can find in the downloads  section.  Currently in beta or at least commit (bc2d2de4321027ca1a8a844997e2d59b8a9d27a0).

An example of a flex application connecting to a server on localhost is below

	<?xml version="1.0" encoding="utf-8"?>
	<s:Application xmlns:fx="http://ns.adobe.com/mxml/2009" 
				   xmlns:s="library://ns.adobe.com/flex/spark" 
				   xmlns:mx="library://ns.adobe.com/flex/mx" minWidth="955" minHeight="600"
				   creationComplete="application1_creationCompleteHandler(event)">
		<s:layout>
			<s:VerticalLayout />
		</s:layout>
		<fx:Declarations>
			<!-- Place non-visual elements (e.g., services, value objects) here -->
		</fx:Declarations>
		<fx:Script>
			<![CDATA[
				import com.pnwrain.flashsocket.FlashSocket;
				import com.pnwrain.flashsocket.events.FlashSocketEvent;

				import mx.controls.Alert;
				import mx.events.FlexEvent;

				[Bindable]
				protected var socket:FlashSocket;

				protected function application1_creationCompleteHandler(event:FlexEvent):void
				{

					socket = new FlashSocket("localhost:8080");
					socket.addEventListener(FlashSocketEvent.CONNECT, onConnect);
					socket.addEventListener(FlashSocketEvent.MESSAGE, onMessage);
					socket.addEventListener(FlashSocketEvent.IO_ERROR, onError);
					socket.addEventListener(FlashSocketEvent.SECURITY_ERROR, onError);

					socket.addEventListener("my other event", myCustomMessageHandler);
				}

				protected function myCustomMessageHandler(event:FlashSocketEvent):void{
					Alert.show('we got a custom event!')	
				}

				protected function onConnect(event:FlashSocketEvent):void {

					clearStatus();

				}

				protected function onError(event:FlashSocketEvent):void {

					setStatus("something went wrong");

				}

				protected function setStatus(msg:String):void{

					status.text = msg;

				}
				protected function clearStatus():void{

					status.text = "";
					this.currentState = "";

				}

				protected function onMessage(event:FlashSocketEvent):void{

					trace('we got message: ' + event.data);
					socket.send({msgdata: event.data},"my other event");

				}

			]]>
		</fx:Script>
		<s:Label id="status" />
		<s:Label id="glabel" />
	</s:Application>

The latest socket.io tested is 0.9.13 (as of March 29, 2013).  You can download that version using NPM easily:
npm install socket.io@0.9.13

In term of node, you ** MUST ** include the following listener:

var socket = io.listen(server, {transports:['flashsocket', 'websocket', 'htmlfile', 'xhr-polling', 'jsonp-polling']});

If you don't include the flashsocket method you will get the following error:

debug - client authorized
info - handshake authorized **** connection token ****
warn - unknown transport: "flashsocket"

Example of node.js server script will look as follow:

var express = require('express'),
    io = require('socket.io'),
	sys = require('sys'),
	types = require('./public/js/types');


var server = express.createServer();
var port = process.env.PORT || 3000;

server.configure( function(){
    server.use(express.static(__dirname + '/public'));
})
server.listen(port, function() {
    console.log('Listening on ' + port);
});

// socket.io 
var socket = io.listen(server, {transports:['flashsocket', 'websocket', 'htmlfile', 'xhr-polling', 'jsonp-polling']});

	
