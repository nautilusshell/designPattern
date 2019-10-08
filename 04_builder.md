

[TOC]

###### 使用场景

1.  信息的发送协议有很多种，其中包括JSON与XML。如何设计发送方式与信息的关系，让代码简洁易于维护？



###### 解决方案

1. 将JSON与XML编码数据的过程抽离出来，通过Builder接口的实现。

2. 定义编码过程接口

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
   }}
   ```

