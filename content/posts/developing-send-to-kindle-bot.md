---
title: "Developing Send-To-Kindle Telegram Bot"
date: 2022-05-19
type: "post"
image: "/images/sendtokindle.webp"
summary: "Lately, I've been using the Kindle app on my tablet a lot and it was always a hustle to send my e-books to my tablet. You need to convert to a proper format, transfer it somehow... meh... So I decided to create a Telegram bot that does all this for you. You send it a book - it appears on your Kindle. Any format supported. That's it - that simple!"
---

![Image](/images/sendtokindle.webp)

Lately, I've been using the Kindle app on my tablet a lot and it was always a hustle to send my e-books to my tablet. You need to convert to a proper format, transfer it somehow... meh...

So I decided to create a Telegram bot that does all this for you. You send it a book - it appears on your Kindle. Any format supported. That's it - that simple!

I decided to use Go as a language because I wanted to practice it more.

## Programmatically converting books to .mobi

It was the main challenge. I did not want to code a converter myself so I started googling other ways to do it. Also, I decided not to go with some online conversion tools with an API as I did not want to rely on them.

So after a while, I stumbled upon the fact that Calibre - popular software for managing e-books - has [CLI tools](https://manual.calibre-ebook.com).

They had everything, that's perfect!

So I started coding.

The conversion itself looks like this. A simple method that accepts input and output paths for an e-book and passes them to the `ebook-convert` CLI Tool.

```go
func convert(in, out string) error {
 cmd := exec.Command("ebook-convert", in, out)
 if err := cmd.Run(); err != nil {
     return err
 }
 if err := cmd.Wait(); err != nil {
     return err
 }
 if _, err := os.Stat(out); errors.Is(err, os.ErrNotExist) {
     return errConversion
 }
 return nil
}
```

## Telegram integration

Up next was telegram integration. That's easy.

Created bot credentials using [BotFather](https://telegram.me/BotFather).

For integration with Telegram, I found a bot framework for Go - [Telebot](https://github.com/tucnak/telebot).

The setup was pretty easy:

```go
func (b *SendToKindleBot) Start() error {
 bot, err := tb.NewBot(tb.Settings{
     Token:  b.Token,
     Poller: &tb.LongPoller{Timeout: 10 * time.Second},
 })
 if err != nil {
     return ErrStartup
 }
 bot.Start()
 return nil
}
```

## Sending via email

I created an email that my bot would use to send e-books.

Then don't forget to add it to [Approved Personal Document Email List](https://www.amazon.com) in your kindle settings.

And the method itself is pretty simple:

```go
func (b *SendToKindleBot) sendFileViaEmail(path string) error {
 msg := email.NewMessage("", "")
 msg.From = mail.Address{Name: "From", Address: b.EmailFrom}
 msg.To = []string{b.EmailTo}

 if err := msg.Attach(path); err != nil {
     return err
 }

 auth := smtp.PlainAuth("", b.EmailFrom, b.Password, b.SMTPHost)
 addr := fmt.Sprintf("%s:%s", b.SMTPHost, b.SMTPPort)
 if err := email.Send(addr, auth, msg); err != nil {
     return err
 }
 return nil
}
```

## Putting things together

So next I wrote all the insides of the telegram bot with the main thing being - a handler that would receive a file and send it to my kindle.

```go
func (b *SendToKindleBot) documentHandler(bot *tb.Bot) func(msg *tb.Message) {
 return func(msg *tb.Message) {
     doc := msg.Document
     nameParts := strings.Split(doc.FileName, ".")
     fileNameWithoutExtension := strings.Join(nameParts[:len(nameParts)-1], "")
     extension := nameParts[len(nameParts)-1]

     originalFilePath := tmpFilesPath + doc.FileName
     if err := bot.Download(&doc.File, originalFilePath); err != nil {
         log.Println("could not download file", err)
         respond(bot, msg, "Sorry. I could not download file")
     }
     defer removeSilently(originalFilePath)

     fileToSend := originalFilePath
     if needToConvert(extension) {
         outputFilePath := tmpFilesPath + fileNameWithoutExtension + ".mobi"
         if err := convert(originalFilePath, outputFilePath); err != nil {
             log.Println("could not convert file", err)
             respond(bot, msg, "Sorry. I could not convert file")
         }
         fileToSend = outputFilePath
         defer removeSilently(outputFilePath)
     }

     if err := b.sendFileViaEmail(fileToSend); err != nil {
         log.Println("could not send file", err)
         respond(bot, msg, "Sorry. I could not send file")
     }
 }
}
```

And registered it in the bot:

```go
...
bot.Handle(tb.OnDocument, b.documentHandler(bot))
...
```

## Creating docker container

I wanted to put this app inside a docker container so I can deploy it easily in DigitalOcean and have it running 24/7.

This came to be a bit tricky. Installation of Calibre Tools was not straightforward. Still, after lots of tries, I came up with the following Dockerfile:

```dockerfile
FROM amd64/ubuntu
COPY --from=golang:1.16.11-bullseye /usr/local/go/ /usr/local/go/
ENV PATH="/usr/local/go/bin:${PATH}"

RUN go env -w GOPROXY=direct GOFLAGS="-insecure"
ENV TZ=Europe/Minsk
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y git && \
    apt-get install wget && \
    apt-get install -y python
RUN apt-get install -y ffmpeg libsm6 libxext6
RUN wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sh /dev/stdin
RUN mkdir -p files
WORKDIR /app

COPY . .

RUN go build

RUN chmod +x ./send-to-kindle-telegram-bot
CMD ["./send-to-kindle-telegram-bot"]
```

## Conclusion

After a lot of testing, I finally got the app up and running on DigitalOcean.

Hope you find this bot useful!

Full code you can find on my GitHub: [https://github.com/michaelfmnk/send-to-kindle-telegram-bot](https://github.com/michaelfmnk/send-to-kindle-telegram-bot)

I'm open to any suggestions, ideas, or collaboration :)

---

©2022 by Mykhailo Fomenko