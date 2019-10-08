

[TOC]

###### 使用场景

业务方通过HTTP库来发送请求。如果希望统计每个HTTP请求的耗时，并打印每个请求的响应结果，但同时又不希望在每处调用都编写额外的代码。那么该如何处理呢？



###### 解决方案

1. 为HTTP库建立一个代理（Proxy或Delegate），代理的接口与HTTP库接口保持一致。由代理内部统一执行耗时统计与日志记录。此模式相当于C语言的Wrapper函数。

2. 定义HTTP Proxy对象

   ```go
   type Message struct {
   	body 		[]byte
   	format 		string
   }
   
   type MessageBuilder interface {
   	SetRecipient(recipient string)
   	SetText(text string)
   	BuildMessage() (*Message, error)
   }
   ```

3. 实现不同的编码过程

   ```go
   
   type JSONMessageBuilder struct{
   	messageRecipient 	 string
   	messageText 		 string
   }
   
   func (j *JSONMessageBuilder) SetRecipient(recipient string) {
   	j.messageRecipient = recipient
   }
   
   func (j *JSONMessageBuilder) SetText(text string) {
   	j.messageText = text
   }
   
   func (j *JSONMessageBuilder) BuildMessage() (*Message, error) {
   	msg := make(map[string]string)
   	msg["recipient"] 	= j.messageRecipient
   	msg["message"] 		= j.messageText
   
   	data, err := json.Marshal(msg)
   	if err != nil {
   		return nil, err
   	}
   
   	return &Message{ body:data, format: "JSON",}, nil
   }
   
   type XMLMessageBuilder struct{
   	messageRecipient 	 string
   	messageText 		 string
   }
   
   func (b *XMLMessageBuilder) SetRecipient(recipient string) {
   	b.messageRecipient = recipient
   }
   
   func (b *XMLMessageBuilder) SetText(text string) {
   	b.messageText = text
   }
   
   func (b *XMLMessageBuilder) BuildMessage() (*Message, error) {
   	type XMLMessage struct {
   		Recipient		string `xml:"recipient"`
   		Text 			string `xml:"body"`
   	}
   
   	m := &XMLMessage{
   		Recipient:  b.messageRecipient,
   		Text:       b.messageText,
   	}
   
   	data, err := xml.Marshal(m)
   	if err != nil {
   		return nil, err
   	}
   
   	return &Message{body: data, format: "XML",}, nil
   }
   
   type Sender struct {
   	Recipient 		string
   	Text 			string
   }
   
   func (s Sender) BuildMessage(builder MessageBuilder) (*Message, error) {
   	builder.SetRecipient(s.Recipient)
   	builder.SetText(s.Text)
   	return builder.BuildMessage()
   }
   ```

