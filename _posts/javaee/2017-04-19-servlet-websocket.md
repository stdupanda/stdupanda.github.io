---
layout: post
title: 使用注解进行 WebSocket 开发
categories: JavaWeb
description: 使用注解进行 WebSocket 开发
keywords: java, web, javaweb, JavaWeb, WebSocket, websocket
---

JavaEE 7 对 WebSocket 的支持，使用 注解开发，Tomcat 7.0.5版本以上，JSR356 规范。

```java
package websocket;

import java.io.IOException;
import java.util.concurrent.CopyOnWriteArrayList;

import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ServerEndpoint("/push_notify/{reqId}")
public class WebsocketDemo {
	private static final Logger log = LoggerFactory.getLogger(WebsocketDemo.class);
	private CopyOnWriteArrayList<WebsocketDemo> list = new CopyOnWriteArrayList<>();

	/**
	 * 返回消息给远程客户端
	 * 
	 * @param message
	 * @throws IOException
	 */
	public void sendMessage(Session session, String message) throws IOException {
		session.getBasicRemote().sendText(message);
		// this.session.getAsyncRemote().sendText(message);
	}

	/**
	 * 打开连接时触发
	 * 
	 * @param reqId
	 * @param session
	 */
	@OnOpen
	public void onOpen(@PathParam("reqId") String reqId, Session session) {
		list.add(this);
		WSMap.put(reqId, session);
		log.info("Websocket Start Connecting(onOpen):" + reqId);
		try {
			sendMessage(session, "hello:" + reqId);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	/**
	 * 收到客户端消息时触发
	 * 
	 * @param reqId
	 * @param message
	 * @return
	 */
	@OnMessage
	public String onMessage(@PathParam("reqId") String reqId, String message) {
		return "Got your message (" + message + ").Thanks !";
	}

	/**
	 * 异常时触发
	 * 
	 * @param reqId
	 * @param userCode
	 * @param session
	 */
	@OnError
	public void onError(@PathParam("reqId") String reqId, Throwable throwable, Session session) {
		list.remove(this);
		WSMap.remove(reqId);
		log.error("Websocket Connection Exception:" + reqId);
		log.error(throwable.getMessage(), throwable);
	}

	/**
	 * 关闭连接时触发
	 * 
	 * @param reqId
	 * @param userCode
	 * @param session
	 */
	@OnClose
	public void onClose(@PathParam("reqId") String reqId, Session session) {
		list.remove(this);
		WSMap.remove(reqId);
		log.info("Websocket Close Connection:" + reqId);
	}
}
```