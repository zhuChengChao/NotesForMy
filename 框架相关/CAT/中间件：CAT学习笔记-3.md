# 中间件：CAT学习笔记-3

> 实习过程中了解到存在调用链监控这种好用的工具，就想稍微系统的学习一下，发现也有相应的视频课程：[Java进阶教程Cat入门](https://www.bilibili.com/video/BV1m64y127f3)，就顺道看了一波，跟着课程内容做了下学习笔记，此为 3/3 篇。

# 4. CAT原理

本章中介绍两部分内容：

- 客户端原理介绍
- 服务端原理介绍

## 4.1 客户端原理

### 4.1.1 客户端设计

#### 架构设计

客户端设计是 CAT 系统设计中最为核心的一个环节，客户端要求是做到 API 简单、高可靠性能，因为监控只是公司核心业务流程一个旁路环节，无论在任何场景下都不能影响业务性能。

CAT 客户端在**收集端数据方面使用 ThreadLocal**（线程局部变量），是线程本地变量，也可以称之为线程本地存储。其实 ThreadLocal 的功用非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，属于 Java 中一种较为特殊的线程绑定机制，每一个线程都可以独立地改变自己的副本，不会和其它线程的副本冲突。

在监控场景下，为用户提供服务都是 Web 容器，比如 Tomcat 或者 Jetty，后端的 RPC 服务端比如 Dubbo 或者 Pigeon，也都是基于线程池来实现的。业务方在处理业务逻辑时基本都是在一个线程内部调用后端服务、数据库、缓存等，将这些数据拿回来再进行业务逻辑封装，最后将结果展示给用户。**所以将所有的监控请求作为一个监控上下文存入线程变量就非常合适**。

![client](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408142.png)

如上图所示，业务执行业务逻辑的时候，就会把此次请求对应的监控存放于**线程上下文中**，存于上下文的其实是一个**监控树**的结构。在最后业务线程执行结束时，将监控对象存入一个**异步内存队列**中，CAT 有个消费线程将队列内的数据异步发送到服务端。

**总结流程如下：**

- 业务线程产生消息，交给消息 Producer，消息 Producer 将消息存放在该业务线程**消息栈**中；
- 业务线程通知消息 Producer 消息结束时，消息 Producer 根据其消息栈产生**消息树**放置在同步消息队列 Message Queue 中；
- 消息上报线程监听消息队列，根据消息树产生最终的消息报文上报 CAT 服务端。

#### API 设计

监控 API 定义往往取决于对监控或者性能分析这个领域的理解，监控和性能分析所针对的场景有如下几种：

- **一段代码的执行时间**，一段代码可以是 URL 的执行耗时，也可以是 SQL 的执行耗时。
- **一段代码的执行次数**，比如 Java 抛出异常记录次数，或者一段逻辑的执行次数。
- **定期执行某段代码**，比如定期上报一些核心指标：JVM 内存、GC 等指标。
- **关键的业务监控指标**，比如监控订单数、交易额、支付成功率等。

在上述领域模型的基础上，CAT 设计自己核心的几个监控对象：**Transaction、Event、Heartbeat、Metric**

一段监控 API 的代码示例如下：

![监控API](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408200.png)

#### 序列化和通信

序列化和通信是整个客户端包括服务端性能里面很关键的一个环节

- CAT 序列化协议是**自定义序列化协议**，自定义序列化协议相比通用序列化协议要高效很多，这个在大规模数据实时处理场景下还是非常有必要的。
- CAT 通信是基于 Netty 来实现的 NIO 的数据传输，Netty 是一个非常好的 NIO 开发框架，在这边就不详细介绍了。

### 4.1.2 核心类分析

![MessageTree](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261408862.png)

CAT 将监控的内容分为了4种：

- **Transaction：适合记录跨越系统边界的程序访问行为，比如远程调用、数据库调用，也适合执行时间较长的业务逻辑监控，Transaction 用来记录一段代码的执行时间和次数**
- **Event：用来记录一件事发生的次数，比如记录系统异常，它和 Transaction 相比缺少了时间的统计，开销比 Transaction 要小**
- **Heartbeat：以一分钟为周期，定期向CAT服务端汇报当前客户端运行时候的一些状态**
- **Metric：用于记录业务指标、指标可能包含对一个指标记录次数、记录平均值、记录总和，业务指标最低统计粒度为1分钟**

使用 4个接口定义他们的行为，对应的实现类命名方式均为 `Default+接口名`。4个接口都继承自 Message 接口，以下是 Message 接口的定义：

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message;

/**
 * <p>
 * Message represents data collected during application runtime. It will be sent to back-end system asynchronous for
 * further processing.
 * </p>
 * <p>
 * <p>
 * Super interface of <code>Event</code>, <code>Heartbeat</code> and <code>Transaction</code>.
 * </p>
 *
 * @author Frankie Wu
 * @see Event, Heartbeat, Transaction
 */
public interface Message {
	public static final String SUCCESS = "0";

	/**
	 * add one or multiple key-value pairs to the message.
	 *
	 * @param keyValuePairs key-value pairs like 'a=1&b=2&...'
	 */
	public void addData(String keyValuePairs);

	/**
	 * add one key-value pair to the message.
	 *
	 * @param key
	 * @param value
	 */
	public void addData(String key, Object value);

	/**
	 * Complete the message construction.
	 */
	public void complete();

	/**
	 * @return key value pairs data
	 */
	public Object getData();

	/**
	 * Message name.
	 *
	 * @return message name
	 */
	public String getName();

	/**
	 * Get the message status.
	 *
	 * @return message status. "0" means success, otherwise error code.
	 */
	public String getStatus();

	/**
	 * Set the message status with exception class name.
	 *
	 * @param e exception.
	 */
	public void setStatus(Throwable e);

	/**
	 * The time stamp the message was created.
	 *
	 * @return message creation time stamp in milliseconds
	 */
	public long getTimestamp();

	public void setTimestamp(long timestamp);

	/**
	 * Message type.
	 * <p>
	 * <p>
	 * Typical message types are:
	 * <ul>
	 * <li>URL: maps to one method of an action</li>
	 * <li>Service: maps to one method of service call</li>
	 * <li>Search: maps to one method of search call</li>
	 * <li>SQL: maps to one SQL statement</li>
	 * <li>Cache: maps to one cache access</li>
	 * <li>Error: maps to java.lang.Throwable (java.lang.Exception and java.lang.Error)</li>
	 * </ul>
	 * </p>
	 *
	 * @return message type
	 */
	public String getType();

	/**
	 * If the complete() method was called or not.
	 *
	 * @return true means the complete() method was called, false otherwise.
	 */
	public boolean isCompleted();

	/**
	 * @return
	 */
	public boolean isSuccess();

	/**
	 * Set the message status.
	 *
	 * @param status message status. "0" means success, otherwise error code.
	 */
	public void setStatus(String status);

	public void setSuccessStatus();
}
```

这个接口中主要用来提供通用性的方法，比如添加数据 addData、设置状态 setStatus等。所以上述提到的四个功能性的接口均继承自它，比如最复杂的 Transaction 接口：

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message;

import java.util.List;

/**
 * <p>
 * <code>Transaction</code> is any interesting unit of work that takes time to complete and may fail.
 * </p>
 * <p>
 * <p>
 * Basically, all data access across the boundary needs to be logged as a <code>Transaction</code> since it may fail and
 * time consuming. For example, URL request, disk IO, JDBC query, search query, HTTP request, 3rd party API call etc.
 * </p>
 * <p>
 * <p>
 * Sometime if A needs call B which is owned by another team, although A and B are deployed together without any
 * physical boundary. To make the ownership clear, there could be some <code>Transaction</code> logged when A calls B.
 * </p>
 * <p>
 * <p>
 * Most of <code>Transaction</code> should be logged in the infrastructure level or framework level, which is
 * transparent to the application.
 * </p>
 * <p>
 * <p>
 * All CAT message will be constructed as a message tree and send to back-end for further analysis, and for monitoring.
 * Only <code>Transaction</code> can be a tree node, all other message will be the tree leaf.　The transaction without
 * other messages nested is an atomic transaction.
 * </p>
 *
 * @author Frankie Wu
 */
public interface Transaction extends Message {
	/**
	 * Add one nested child message to current transaction.
	 *
	 * @param message to be added
	 */
	public Transaction addChild(Message message);

	/**
	 * Get all children message within current transaction.
	 * <p>
	 * <p>
	 * Typically, a <code>Transaction</code> can nest other <code>Transaction</code>s, <code>Event</code>s and
	 * <code>Heartbeat</code> s, while an <code>Event</code> or <code>Heartbeat</code> can't nest other messages.
	 * </p>
	 *
	 * @return all children messages, empty if there is no nested children.
	 */
	public List<Message> getChildren();

	/**
	 * How long the transaction took from construction to complete. Time unit is microsecond.
	 *
	 * @return duration time in microsecond
	 */
	public long getDurationInMicros();

	/**
	 * How long the transaction took from construction to complete. Time unit is millisecond.
	 *
	 * @return duration time in millisecond
	 */
	public long getDurationInMillis();

	/**
	 * set duration in millisecond.
	 *
	 * @return duration time in millisecond
	 */
	public void setDurationInMillis(long durationInMills);

	/**
	 * Has children or not. An atomic transaction does not have any children message.
	 *
	 * @return true if child exists, else false.
	 */
	public boolean hasChildren();

	/**
	 * Check if the transaction is stand-alone or belongs to another one.
	 *
	 * @return true if it's an root transaction.
	 */
	public boolean isStandalone();
}
```

这个接口继承自 Message 接口。扩展了例如添加子节点addChild、设置执行时间setDurationInMillis 等。最后在 DefaultTransaction 实现类中实现这些方法即可，DefaultTransaction 会继承自 AbstractMessage，这是一个抽象类，实现了 Message 定义的方法：

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message.internal;

import java.nio.charset.Charset;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufAllocator;

import com.dianping.cat.message.Message;
import com.dianping.cat.message.spi.codec.PlainTextMessageCodec;

public abstract class AbstractMessage implements Message {
    protected String m_status = "unset";

    private String m_type;

    private String m_name;

    private long m_timestampInMillis;

    private CharSequence m_data;

    private boolean m_completed;

    public AbstractMessage(String type, String name) {
        m_type = String.valueOf(type);
        m_name = String.valueOf(name);
        m_timestampInMillis = MilliSecondTimer.currentTimeMillis();
    }

    @Override
    public void addData(String keyValuePairs) {
        if (m_data == null) {
            m_data = keyValuePairs;
        } else if (m_data instanceof StringBuilder) {
            ((StringBuilder) m_data).append('&').append(keyValuePairs);
        } else {
            StringBuilder sb = new StringBuilder(m_data.length() + keyValuePairs.length() + 16);

            sb.append(m_data).append('&');
            sb.append(keyValuePairs);
            m_data = sb;
        }
    }

    @Override
    public void addData(String key, Object value) {
        if (m_data instanceof StringBuilder) {
            ((StringBuilder) m_data).append('&').append(key).append('=').append(value);
        } else {
            String str = String.valueOf(value);
            int old = m_data == null ? 0 : m_data.length();
            StringBuilder sb = new StringBuilder(old + key.length() + str.length() + 16);

            if (m_data != null) {
                sb.append(m_data).append('&');
            }

            sb.append(key).append('=').append(str);
            m_data = sb;
        }
    }

    @Override
    public CharSequence getData() {
        if (m_data == null) {
            return "";
        } else {
            return m_data;
        }
    }

    public void setData(String str) {
        m_data = str;
    }

    @Override
    public String getName() {
        return m_name;
    }

    public void setName(String name) {
        m_name = name;
    }

    @Override
    public String getStatus() {
        return m_status;
    }

    @Override
    public void setStatus(Throwable e) {
        m_status = e.getClass().getName();
    }

    @Override
    public long getTimestamp() {
        return m_timestampInMillis;
    }

    @Override
    public void setTimestamp(long timestamp) {
        m_timestampInMillis = timestamp;
    }

    @Override
    public String getType() {
        return m_type;
    }

    public void setType(String type) {
        m_type = type;
    }

    @Override
    public boolean isCompleted() {
        return m_completed;
    }

    public void setCompleted(boolean completed) {
        m_completed = completed;
    }

    @Override
    public boolean isSuccess() {
        return Message.SUCCESS.equals(m_status);
    }

    @Override
    public void setStatus(String status) {
        m_status = status;
    }

    @Override
    public String toString() {
        PlainTextMessageCodec codec = new PlainTextMessageCodec();
        ByteBuf buf = ByteBufAllocator.DEFAULT.buffer();

        codec.encodeMessage(this, buf);
        codec.reset();
        return buf.toString(Charset.forName("utf-8"));
    }

    @Override
    public void setSuccessStatus() {
        m_status = SUCCESS;
    }

}
```

这样一来 DefaultTransaction 只需要实现 Transaction 定义的方法即可，当然如果有部分方法的逻辑比较特殊，可以选择性的覆盖:

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message.internal;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Message;
import com.dianping.cat.message.Transaction;
import com.dianping.cat.message.spi.MessageManager;

public class DefaultTransaction extends AbstractMessage implements Transaction {
	private long m_durationInMicro = -1; // must be less than 0

	private List<Message> m_children;

	private MessageManager m_manager;

	private boolean m_standalone;

	private long m_durationStart;

	public DefaultTransaction(String type, String name) {
		super(type, name);
		m_durationStart = System.nanoTime();
	}

	public DefaultTransaction(String type, String name, MessageManager manager) {
		super(type, name);

		m_manager = manager;
		m_standalone = true;
		m_durationStart = System.nanoTime();
	}

	@Override
	public DefaultTransaction addChild(Message message) {
		if (m_children == null) {
			m_children = new ArrayList<Message>();
		}

		if (message != null) {
			m_children.add(message);
		} else {
			Cat.logError(new Exception("null child message"));
		}
		return this;
	}

	@Override
	public void complete() {
		try {
			if (isCompleted()) {
				// complete() was called more than once
				DefaultEvent event = new DefaultEvent("cat", "BadInstrument");

				event.setStatus("TransactionAlreadyCompleted");
				event.complete();
				addChild(event);
			} else {
				if (m_durationInMicro == -1) {
					m_durationInMicro = (System.nanoTime() - m_durationStart) / 1000L;
				}
				setCompleted(true);
				if (m_manager != null) {
					m_manager.end(this);
				}
			}
		} catch (Exception e) {
			// ignore
		}
	}

	@Override
	public List<Message> getChildren() {
		if (m_children == null) {
			return Collections.emptyList();
		}

		return m_children;
	}

	@Override
	public long getDurationInMicros() {
		if (m_durationInMicro >= 0) {
			return m_durationInMicro;
		} else { // if it's not completed explicitly
			long duration = 0;
			int len = m_children == null ? 0 : m_children.size();

			if (len > 0) {
				Message lastChild = m_children.get(len - 1);

				if (lastChild instanceof Transaction) {
					DefaultTransaction trx = (DefaultTransaction) lastChild;

					duration = (trx.getTimestamp() - getTimestamp()) * 1000L;
				} else {
					duration = (lastChild.getTimestamp() - getTimestamp()) * 1000L;
				}
			}

			return duration;
		}
	}

	public void setDurationInMicros(long duration) {
		m_durationInMicro = duration;
	}

	@Override
	public long getDurationInMillis() {
		return getDurationInMicros() / 1000L;
	}

	@Override
	public void setDurationInMillis(long duration) {
		m_durationInMicro = duration * 1000L;
	}

	protected MessageManager getManager() {
		return m_manager;
	}

	@Override
	public boolean hasChildren() {
		return m_children != null && m_children.size() > 0;
	}

	@Override
	public boolean isStandalone() {
		return m_standalone;
	}

	public void setStandalone(boolean standalone) {
		m_standalone = standalone;
	}

	public void setDurationStart(long durationStart) {
		m_durationStart = durationStart;
	}

	@Override
	public void setStatus(Throwable e) {
		m_status = e.getClass().getName();
		m_manager.getThreadLocalMessageTree().setDiscard(false);
	}
}
```

上面的 setStatus 就覆盖掉了抽象类中定义的方法。

最后所有的数据都会放到 DefaultMessageTree 中：

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message.spi.internal;

import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.List;

import com.dianping.cat.message.io.BufReleaseHelper;
import io.netty.buffer.ByteBuf;

import com.dianping.cat.Cat;
import com.dianping.cat.message.Event;
import com.dianping.cat.message.Heartbeat;
import com.dianping.cat.message.Message;
import com.dianping.cat.message.Metric;
import com.dianping.cat.message.Transaction;
import com.dianping.cat.message.internal.MessageId;
import com.dianping.cat.message.spi.MessageTree;
import com.dianping.cat.message.spi.codec.PlainTextMessageCodec;

public class DefaultMessageTree implements MessageTree {

	private ByteBuf m_buf;

	private String m_domain;

	private String m_hostName;

	private String m_ipAddress;

	private Message m_message;

	private String m_messageId;

	private String m_parentMessageId;

	private String m_rootMessageId;

	private String m_sessionToken;

	private String m_threadGroupName;

	private String m_threadId;

	private String m_threadName;

	private MessageId m_formatMessageId;

	private boolean m_discard = true;

	private boolean m_processLoss = false;

	private boolean m_hitSample = false;

	private List<Event> events = new ArrayList<Event>();

	private List<Transaction> transactions = new ArrayList<Transaction>();

	private List<Heartbeat> heartbeats = new ArrayList<Heartbeat>();

	private List<Metric> metrics = new ArrayList<Metric>();

	@Override
	public boolean canDiscard() {
		return m_discard;
	}

	@Override
	public MessageTree copy() {
		MessageTree tree = new DefaultMessageTree();

		tree.setDomain(m_domain);
		tree.setHostName(m_hostName);
		tree.setIpAddress(m_ipAddress);
		tree.setMessageId(m_messageId);
		tree.setParentMessageId(m_parentMessageId);
		tree.setRootMessageId(m_rootMessageId);
		tree.setSessionToken(m_sessionToken);
		tree.setThreadGroupName(m_threadGroupName);
		tree.setThreadId(m_threadId);
		tree.setThreadName(m_threadName);
		tree.setMessage(m_message);
		tree.setDiscardPrivate(m_discard);
		tree.setHitSample(m_hitSample);
		return tree;
	}

	public List<Event> findOrCreateEvents() {
		if (events == null) {
			events = new ArrayList<Event>();
		}
		return events;
	}

	public List<Heartbeat> findOrCreateHeartbeats() {
		if (heartbeats == null) {
			heartbeats = new ArrayList<Heartbeat>();
		}
		return heartbeats;
	}

	public List<Metric> findOrCreateMetrics() {
		if (metrics == null) {
			metrics = new ArrayList<Metric>();
		}
		return metrics;
	}

	public List<Transaction> findOrCreateTransactions() {
		if (transactions == null) {
			transactions = new ArrayList<Transaction>();
		}
		return transactions;
	}

	public MessageTree copyForTest() {
		ByteBuf buf = null;
		try {
			PlainTextMessageCodec codec = new PlainTextMessageCodec();
			buf = codec.encode(this);
			//buf.readInt(); // get rid of length

			return codec.decode(buf);
		} catch (Exception ex) {
			Cat.logError(ex);
		}

		return null;
	}

	public void clearMessageList() {
		if (transactions != null) {
			transactions.clear();
		}

		if (events != null) {
			events.clear();
		}

		if (heartbeats != null) {
			heartbeats.clear();
		}

		if (metrics != null) {
			metrics.clear();
		}
	}

	public ByteBuf getBuffer() {
		return m_buf;
	}

	public void setBuffer(ByteBuf buf) {
		m_buf = buf;
	}

	@Override
	public String getDomain() {
		return m_domain;
	}

	@Override
	public void setDomain(String domain) {
		m_domain = domain;
	}

	public List<Event> getEvents() {
		return events;
	}

	public MessageId getFormatMessageId() {
		if (m_formatMessageId == null) {
			m_formatMessageId = MessageId.parse(m_messageId);
		}

		return m_formatMessageId;
	}

	public void setFormatMessageId(MessageId formatMessageId) {
		m_formatMessageId = formatMessageId;
	}

	public List<Heartbeat> getHeartbeats() {
		return heartbeats;
	}

	@Override
	public String getHostName() {
		return m_hostName;
	}

	@Override
	public void setHostName(String hostName) {
		m_hostName = hostName;
	}

	@Override
	public String getIpAddress() {
		return m_ipAddress;
	}

	@Override
	public void setIpAddress(String ipAddress) {
		m_ipAddress = ipAddress;
	}

	@Override
	public String getSessionToken() {
		return m_sessionToken;
	}

	@Override
	public void setSessionToken(String sessionToken) {
		m_sessionToken = sessionToken;
	}

	@Override
	public Message getMessage() {
		return m_message;
	}

	@Override
	public void setMessage(Message message) {
		m_message = message;
	}

	@Override
	public String getMessageId() {
		return m_messageId;
	}

	@Override
	public void setMessageId(String messageId) {
		if (messageId != null && messageId.length() > 0) {
			m_messageId = messageId;
		}
	}

	public List<Metric> getMetrics() {
		return metrics;
	}

	@Override
	public String getParentMessageId() {
		return m_parentMessageId;
	}

	@Override
	public void setParentMessageId(String parentMessageId) {
		if (parentMessageId != null && parentMessageId.length() > 0) {
			m_parentMessageId = parentMessageId;
		}
	}

	@Override
	public String getRootMessageId() {
		return m_rootMessageId;
	}

	@Override
	public void setRootMessageId(String rootMessageId) {
		if (rootMessageId != null && rootMessageId.length() > 0) {
			m_rootMessageId = rootMessageId;
		}
	}

	@Override
	public String getThreadGroupName() {
		return m_threadGroupName;
	}

	@Override
	public void setThreadGroupName(String threadGroupName) {
		m_threadGroupName = threadGroupName;
	}

	@Override
	public String getThreadId() {
		return m_threadId;
	}

	@Override
	public void setThreadId(String threadId) {
		m_threadId = threadId;
	}

	@Override
	public String getThreadName() {
		return m_threadName;
	}

	@Override
	public void setThreadName(String threadName) {
		m_threadName = threadName;
	}

	public List<Transaction> getTransactions() {
		return transactions;
	}

	@Override
	public boolean isProcessLoss() {
		return m_processLoss;
	}

	@Override
	public void setProcessLoss(boolean loss) {
		m_processLoss = loss;
	}

	public void setDiscard(boolean discard) {
		m_discard = discard;
	}

	@Override
	public boolean isHitSample() {
		return m_hitSample;
	}

	@Override
	public void setHitSample(boolean hitSample) {
		m_hitSample = hitSample;
	}

	public void setDiscardPrivate(boolean discard) {
		m_discard = discard;
	}

	@Override
	public String toString() {
		ByteBuf buf = null;
		String result = "";
		try {
			PlainTextMessageCodec codec = new PlainTextMessageCodec();
			buf = codec.encode(this);
			buf.readInt(); // get rid of length
			result = buf.toString(Charset.forName("utf-8"));
		} catch (Exception ex) {
			Cat.logError(ex);
		} finally {
			if (buf != null) {
				BufReleaseHelper.release(buf);
			}
		}

        return result;
    }
}
```

这里用了四个 ArrayList 来存放对应的数据：

```java
private List<Event> events = new ArrayList<Event>();
private List<Transaction> transactions = new ArrayList<Transaction>();
private List<Heartbeat> heartbeats = new ArrayList<Heartbeat>();
private List<Metric> metrics = new ArrayList<Metric>();
```

### 4.1.3 流程分析

#### 启动

懒加载创建 Cat 客户端对象：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409469.png)

读取 client.xml：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409925.png)

加载模块：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409321.png)

#### 创建Message

创建一个新的 Transaction:

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409634.png)

创建上下文：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409238.png)

添加 Transaction 到上下文中:

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409560.png)

添加 Transaction 到 DefaultMessageTree 中:

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409036.png)

关闭 Transaction:

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409725.png)

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409426.png)

这里需要介绍一下，消息进入到上下文之后，是通过栈的方式来存储的：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409071.png)

Transaction 之间是有引用的，因此在 end 方法中只需要将第一个 Transaction（封装在MessageTree 中）通过 MessageManager 来 flush，在拼接消息时可以根据这个引用关系来找到所有的 Transaction 。所以来看代码：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409467.png)

#### 发送数据

首先获取到发送类的对象，调用其方法进行发送：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409046.png)

发送时是经典的**生产者-消费者模型**，生产者只需要向队列中放入数据，消费者监听队列，获取数据并发送：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409601.png)

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409675.png)

消费者线程拉取消息：

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409264.png)

使用自定义的序列化方式进行序列化，最后使用 Netty 发送数据:

![](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409677.png)

Cat 使用了自定义的序列化方式：

```java
/*
 * Copyright (c) 2011-2018, Meituan Dianping. All Rights Reserved.
 *
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.dianping.cat.message.spi.codec;

import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.Stack;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.PooledByteBufAllocator;

import com.dianping.cat.message.Event;
import com.dianping.cat.message.Heartbeat;
import com.dianping.cat.message.Message;
import com.dianping.cat.message.Metric;
import com.dianping.cat.message.Trace;
import com.dianping.cat.message.Transaction;
import com.dianping.cat.message.internal.DefaultEvent;
import com.dianping.cat.message.internal.DefaultHeartbeat;
import com.dianping.cat.message.internal.DefaultMetric;
import com.dianping.cat.message.internal.DefaultTrace;
import com.dianping.cat.message.internal.DefaultTransaction;
import com.dianping.cat.message.spi.MessageCodec;
import com.dianping.cat.message.spi.MessageTree;
import com.dianping.cat.message.spi.internal.DefaultMessageTree;

public class NativeMessageCodec implements MessageCodec {

	public static final String ID = "NT1"; // native message tree version 1

	@Override
	public MessageTree decode(ByteBuf buf) {
		buf.readInt(); // read the length of the message tree

		DefaultMessageTree tree = new DefaultMessageTree();
		Context ctx = new Context(tree);
		Codec.HEADER.decode(ctx, buf);
		Message msg = decodeMessage(ctx, buf);

		tree.setMessage(msg);
		tree.setBuffer(buf);

		return tree;
	}

	private Message decodeMessage(Context ctx, ByteBuf buf) {
		Message msg = null;

		while (buf.readableBytes() > 0) {
			char ch = ctx.readId(buf);

			switch (ch) {
			case 't':
				Codec.TRANSACTION_START.decode(ctx, buf);
				break;
			case 'T':
				msg = Codec.TRANSACTION_END.decode(ctx, buf);
				break;
			case 'E':
				Message e = Codec.EVENT.decode(ctx, buf);

				ctx.addChild(e);
				break;
			case 'M':
				Message m = Codec.METRIC.decode(ctx, buf);

				ctx.addChild(m);
				break;
			case 'H':
				Message h = Codec.HEARTBEAT.decode(ctx, buf);

				ctx.addChild(h);
				break;
			case 'L':
				Message l = Codec.TRACE.decode(ctx, buf);

				ctx.addChild(l);
				break;
			default:
				throw new RuntimeException(String.format("Unsupported message type(%s).", ch));
			}
		}

		if (msg == null) {
			msg = ctx.getMessageTree().getMessage();
		}

		return msg;
	}

	@Override
	public ByteBuf encode(MessageTree tree) {
		ByteBuf buf = PooledByteBufAllocator.DEFAULT.buffer(4 * 1024);

		try {
			Context ctx = new Context(tree);

			buf.writeInt(0); // place-holder

			Codec.HEADER.encode(ctx, buf, null);

			Message msg = tree.getMessage();

			if (msg != null) {
				encodeMessage(ctx, buf, msg);
			}
			int readableBytes = buf.readableBytes();

			buf.setInt(0, readableBytes - 4); // reset the message size

			return buf;
		} catch (RuntimeException e) {
			buf.release();

			throw e;
		}
	}

	private void encodeMessage(Context ctx, ByteBuf buf, Message msg) {
		if (msg instanceof Transaction) {
			Transaction transaction = (Transaction) msg;
			List<Message> children = transaction.getChildren();

			Codec.TRANSACTION_START.encode(ctx, buf, msg);

			for (Message child : children) {
				if (child != null) {
					encodeMessage(ctx, buf, child);
				}
			}

			Codec.TRANSACTION_END.encode(ctx, buf, msg);
		} else if (msg instanceof Event) {
			Codec.EVENT.encode(ctx, buf, msg);
		} else if (msg instanceof Metric) {
			Codec.METRIC.encode(ctx, buf, msg);
		} else if (msg instanceof Heartbeat) {
			Codec.HEARTBEAT.encode(ctx, buf, msg);
		} else if (msg instanceof Trace) {
			Codec.TRACE.encode(ctx, buf, msg);
		} else {
			throw new RuntimeException(String.format("Unsupported message(%s).", msg));
		}
	}

	@Override
	public void reset() {
	}

	enum Codec {
		HEADER {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				MessageTree tree = ctx.getMessageTree();
				String version = ctx.getVersion(buf);

				if (ID.equals(version)) {
					tree.setDomain(ctx.readString(buf));
					tree.setHostName(ctx.readString(buf));
					tree.setIpAddress(ctx.readString(buf));
					tree.setThreadGroupName(ctx.readString(buf));
					tree.setThreadId(ctx.readString(buf));
					tree.setThreadName(ctx.readString(buf));
					tree.setMessageId(ctx.readString(buf));
					tree.setParentMessageId(ctx.readString(buf));
					tree.setRootMessageId(ctx.readString(buf));
					tree.setSessionToken(ctx.readString(buf));
				} else {
					throw new RuntimeException(String.format("Unrecognized version(%s) for binary message codec!", version));
				}

				return null;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				MessageTree tree = ctx.getMessageTree();

				ctx.writeVersion(buf, ID);
				ctx.writeString(buf, tree.getDomain());
				ctx.writeString(buf, tree.getHostName());
				ctx.writeString(buf, tree.getIpAddress());
				ctx.writeString(buf, tree.getThreadGroupName());
				ctx.writeString(buf, tree.getThreadId());
				ctx.writeString(buf, tree.getThreadName());
				ctx.writeString(buf, tree.getMessageId());
				ctx.writeString(buf, tree.getParentMessageId());
				ctx.writeString(buf, tree.getRootMessageId());
				ctx.writeString(buf, tree.getSessionToken());
			}
		},

		TRANSACTION_START {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				long timestamp = ctx.readTimestamp(buf);
				String type = ctx.readString(buf);
				String name = ctx.readString(buf);

				if ("System".equals(type) && name.startsWith("UploadMetric")) {
					name = "UploadMetric";
				}

				DefaultTransaction t = new DefaultTransaction(type, name);

				t.setTimestamp(timestamp);
				ctx.pushTransaction(t);

				MessageTree tree = ctx.getMessageTree();
				if (tree instanceof DefaultMessageTree) {
					tree.getTransactions().add(t);
				}

				return t;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				ctx.writeId(buf, 't');
				ctx.writeTimestamp(buf, msg.getTimestamp());
				ctx.writeString(buf, msg.getType());
				ctx.writeString(buf, msg.getName());
			}
		},

		TRANSACTION_END {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				String status = ctx.readString(buf);
				String data = ctx.readString(buf);
				long durationInMicros = ctx.readDuration(buf);
				DefaultTransaction t = ctx.popTransaction();

				t.setStatus(status);
				t.addData(data);
				t.setDurationInMicros(durationInMicros);
				return t;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				Transaction t = (Transaction) msg;

				ctx.writeId(buf, 'T');
				ctx.writeString(buf, msg.getStatus());
				ctx.writeString(buf, msg.getData().toString());
				ctx.writeDuration(buf, t.getDurationInMicros());
			}
		},

		EVENT {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				long timestamp = ctx.readTimestamp(buf);
				String type = ctx.readString(buf);
				String name = ctx.readString(buf);
				String status = ctx.readString(buf);
				String data = ctx.readString(buf);
				DefaultEvent e = new DefaultEvent(type, name);

				e.setTimestamp(timestamp);
				e.setStatus(status);
				e.addData(data);

				MessageTree tree = ctx.getMessageTree();
				if (tree instanceof DefaultMessageTree) {
					tree.getEvents().add(e);
				}

				return e;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				ctx.writeId(buf, 'E');
				ctx.writeTimestamp(buf, msg.getTimestamp());
				ctx.writeString(buf, msg.getType());
				ctx.writeString(buf, msg.getName());
				ctx.writeString(buf, msg.getStatus());
				ctx.writeString(buf, msg.getData().toString());
			}
		},

		METRIC {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				long timestamp = ctx.readTimestamp(buf);
				String type = ctx.readString(buf);
				String name = ctx.readString(buf);
				String status = ctx.readString(buf);
				String data = ctx.readString(buf);
				DefaultMetric m = new DefaultMetric(type, name);

				m.setTimestamp(timestamp);
				m.setStatus(status);
				m.addData(data);

				MessageTree tree = ctx.getMessageTree();
				if (tree instanceof DefaultMessageTree) {
					tree.getMetrics().add(m);
				}

				return m;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				ctx.writeId(buf, 'M');
				ctx.writeTimestamp(buf, msg.getTimestamp());
				ctx.writeString(buf, msg.getType());
				ctx.writeString(buf, msg.getName());
				ctx.writeString(buf, msg.getStatus());
				ctx.writeString(buf, msg.getData().toString());
			}
		},

		HEARTBEAT {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				long timestamp = ctx.readTimestamp(buf);
				String type = ctx.readString(buf);
				String name = ctx.readString(buf);
				String status = ctx.readString(buf);
				String data = ctx.readString(buf);
				DefaultHeartbeat h = new DefaultHeartbeat(type, name);

				h.setTimestamp(timestamp);
				h.setStatus(status);
				h.addData(data);

				MessageTree tree = ctx.getMessageTree();
				if (tree instanceof DefaultMessageTree) {
					tree.getHeartbeats().add(h);
				}

				return h;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				ctx.writeId(buf, 'H');
				ctx.writeTimestamp(buf, msg.getTimestamp());
				ctx.writeString(buf, msg.getType());
				ctx.writeString(buf, msg.getName());
				ctx.writeString(buf, msg.getStatus());
				ctx.writeString(buf, msg.getData().toString());
			}
		},

		TRACE {
			@Override
			protected Message decode(Context ctx, ByteBuf buf) {
				long timestamp = ctx.readTimestamp(buf);
				String type = ctx.readString(buf);
				String name = ctx.readString(buf);
				String status = ctx.readString(buf);
				String data = ctx.readString(buf);
				DefaultTrace t = new DefaultTrace(type, name);

				t.setTimestamp(timestamp);
				t.setStatus(status);
				t.addData(data);
				return t;
			}

			@Override
			protected void encode(Context ctx, ByteBuf buf, Message msg) {
				ctx.writeId(buf, 'L');
				ctx.writeTimestamp(buf, msg.getTimestamp());
				ctx.writeString(buf, msg.getType());
				ctx.writeString(buf, msg.getName());
				ctx.writeString(buf, msg.getStatus());
				ctx.writeString(buf, msg.getData().toString());
			}
		};

		protected abstract Message decode(Context ctx, ByteBuf buf);

		protected abstract void encode(Context ctx, ByteBuf buf, Message msg);
	}

	private static class Context {
		private static Charset UTF8 = Charset.forName("UTF-8");

		private MessageTree m_tree;

		private Stack<DefaultTransaction> m_parents = new Stack<DefaultTransaction>();

		private byte[] m_data = new byte[256];

		public Context(MessageTree tree) {
			m_tree = tree;
		}

		public void addChild(Message msg) {
			if (!m_parents.isEmpty()) {
				m_parents.peek().addChild(msg);
			} else {
				m_tree.setMessage(msg);
			}
		}

		public MessageTree getMessageTree() {
			return m_tree;
		}

		public String getVersion(ByteBuf buf) {
			byte[] data = new byte[3];

			buf.readBytes(data);
			return new String(data);
		}

		public DefaultTransaction popTransaction() {
			return m_parents.pop();
		}

		public void pushTransaction(DefaultTransaction t) {
			if (!m_parents.isEmpty()) {
				m_parents.peek().addChild(t);
			}

			m_parents.push(t);
		}

		public long readDuration(ByteBuf buf) {
			return readVarint(buf, 64);
		}

		public char readId(ByteBuf buf) {
			return (char) buf.readByte();
		}

		public String readString(ByteBuf buf) {
			int len = (int) readVarint(buf, 32);

			if (len == 0) {
				return "";
			} else if (len > m_data.length) {
				m_data = new byte[len];
			}

			buf.readBytes(m_data, 0, len);
			return new String(m_data, 0, len, StandardCharsets.UTF_8);
		}

		public long readTimestamp(ByteBuf buf) {
			return readVarint(buf, 64);
		}

		protected long readVarint(ByteBuf buf, int length) {
			int shift = 0;
			long result = 0;

			while (shift < length) {
				final byte b = buf.readByte();
				result |= (long) (b & 0x7F) << shift;
				if ((b & 0x80) == 0) {
					return result;
				}
				shift += 7;
			}

			throw new RuntimeException("Malformed variable int " + length + "!");
		}

		public void writeDuration(ByteBuf buf, long duration) {
			writeVarint(buf, duration);
		}

		public void writeId(ByteBuf buf, char id) {
			buf.writeByte(id);
		}

		public void writeString(ByteBuf buf, String str) {
			if (str == null || str.length() == 0) {
				writeVarint(buf, 0);
			} else {
				byte[] data = str.getBytes(UTF8);

				writeVarint(buf, data.length);
				buf.writeBytes(data);
			}
		}

		public void writeTimestamp(ByteBuf buf, long timestamp) {
			writeVarint(buf, timestamp); // TODO use relative value of root message timestamp
		}

		private void writeVarint(ByteBuf buf, long value) {
			while (true) {
				if ((value & ~0x7FL) == 0) {
					buf.writeByte((byte) value);
					return;
				} else {
					buf.writeByte(((byte) value & 0x7F) | 0x80);
					value >>>= 7;
				}
			}
		}

		public void writeVersion(ByteBuf buf, String version) {
			buf.writeBytes(version.getBytes());
		}
	}

}
```

根据不同的数据类型，进行写入即可。

## 4.2 服务端原理

### 4.2.1 架构设计

单机的 consumer 架构设计如下：

![img](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409236.png)

如上图，CAT 服务端在整个实时处理中，**基本上实现了全异步化处理**。

- 消息接受是基于 Netty 的 NIO 实现。
- 消息接受到服务端就存放内存队列，然后程序**开启一个线程会消费这个消息做消息分发**。
- 每个消息都会有一批线程并发消费各自队列的数据，以做到消息处理的隔离。
- 消息存储是先存入本地磁盘，然后异步上传到 HDFS 文件，这也避免了强依赖 HDFS。

当某个报表处理器处理来不及时候，比如 Transaction 报表处理比较慢，可以通过配置支持开启多个Transaction 处理线程，**并发消费消息**。

![img](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409858.png)

### 4.2.2 消息ID设计

CAT 每个消息都有一个唯一的 ID，这个 ID 在客户端生成，后续都通过这个 ID 在进行消息内容的查找。典型的 RPC 消息串起来的问题，比如A调用B的时候，在A这端生成一个 Message-ID，在A调用B的过程中，将 Message-ID 作为调用传递到B端，在B执行过程中，B用 context 传递的 Message-ID 作为当前监控消息的Message-ID。

CAT消息的 Message-ID 格式 ShopWeb-0a010680-375030-2，CAT消息一共分为四段：

- 第一段是应用名 shop-web。
- 第二段是当前这台机器的IP的16进制格式，0a010680表示10.1.6.108。
- 第三段的375030，是系统当前时间除以小时得到的整点数。
- 第四段的2，是表示当前这个客户端在当前小时的顺序递增号。

### 4.2.3 存储数据设计

消息存储是 CAT 最有挑战的部分。关键问题是消息数量多且大，目前美团每天处理消息1000亿左右，大小大约100TB，单物理机高峰期每秒要处理100MB左右的流量。CAT服务端基于此流量做实时计算，还需要将这些数据压缩后写入磁盘。

整体存储结构如下图：

![img](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261409722.png)

CAT 在写数据一份是 Index 文件，一份是 Data 文件.

- Data 文件是分段 GZIP 压缩，每个分段大小小于 64K，这样可以用 16bits 可以表示一个最大分段地址。
- 一个 Message-ID 都用需要 48bits 的大小来存索引，索引根据 Message-ID 的第四段来确定索引的位置，比如消息 Message-ID 为 ShopWeb-0a010680-375030-2，这条消息ID对应的索引位置为 2*48bits 的位置。
- 48bits 前面 32bits 存数据文件的块偏移地址，后面 16bits 存数据文件解压之后的块内地址偏移。
- CAT 读取消息的时候，首先根据 Message-ID 的前面三段确定唯一的索引文件，再根据 Message-ID 第四段确定此 Message-ID 索引位置，根据索引文件的48bits 读取数据文件的内容，然后将数据文件进行 GZIP 解压，再根据块内偏移地址读取出真正的消息内容。
