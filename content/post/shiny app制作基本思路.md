---
title: Shiny app的基本制作思路
author: 'pauke'
date: "2018-12-18"
categories:
  - workflow
  - 汉语
tags: ["R","shiny","app"]
---


Shiny是基于R的实时计算**服务器（serve）**，并通过CSS，htmlwidge，Javascript来进行拓展的web **交互界面（UI）**展现的构造工具包。

R作为一种以本地会话（local session）为主要使用场景的语言，交互性、可嵌入性和自动化一直是其软肋。Rstudio希望发展基于R构建BI工具，就需要将本地的会话和线上的展示交互结合。因此，这也引出了shiny为回应以上需求，而在结构设计上着墨的三个根本要素：服务器（Serve）、交互界面（UI）和反应连结（Reactivity）。

这也可以引出shiny设计的一个根本思路：反应表达式（reactive expression）。最简洁的理解反应式表达的的示例：
```r
input values => R code => output values/result
```

> 当表达式开始执行的时候，将会自动跟踪读取到的反应值以及调用的其他反应表达式。如果反应表达式所依赖的反应值和反应表达式发生了改变，那么该反应表达式的返回值也应该变化，改变一个反应值会自动引发依赖于它的反应表达式重新执行。
> ——[shiny中文教程](http://yanping.me/shiny-tutorial/#reactivity)

具体而言，shiny的构成组件主要是这个样子：

{{<mermaid align="left">}}
graph LR
subgraph 用户界面UI
i["输入(input$x)<br> reactiveValues() <br>*Input()"]  
o("输出(output$y) <br> render*()")
ob("触发展示(Trigger) <br> observe()<br>observeEvent()")
end
subgraph 服务器serve
is("表达抑制(prevent)<br>isolate()")
i --> e{"表达(expression)<br>reactive()"}
i --> ob
e --> is
e--> o
i -->  d{"延迟表达 <br> (delay reaction)<br>eventRactive()"}
d --> e
end
{{< /mermaid >}}

- 此处的serve和ui不等同于shiny中实际的serve和ui函数，仅是指对于app用户来说的最终呈现情况



基于以上对于shiny设计思路的介绍，就能容易理解shiny代码的基本结构，ui部分（对象值），server部分（函数）以及app结合部分（对象值）。

```sql
library(shiny)
ui #UI部分 <- fluidPage( 
numericInput(inputId = "n","Sample size", value = 25),
plotOutput(outputId = "hist")
)
server #服务器部分 <- function(input, output) { 
output$hist <- renderPlot({
            hist(rnorm(input$n))
           })
}

shinyApp(ui = ui, server = server) #二者结合为shiny
```

首先，最基本的问题点是shiny app最初的动机需求。这个初始的需求纲要不要求全备，但希望应该对以下几个点有一定的考虑：

* 数据来源（自带数据、虚拟数据、用户上传数据等）
* 交互输入（点选、键入、拖拽等）
* 大致呈现方式类型（图、表、文字等）
* 交互的数据纬度（交互涉及的数据字段和性质等）

对此有了基本思路之后，就是具体实现的层面。以上纲要也将在具体实现过程中指导具体过程，同时也会在考虑具体实现层面时进一步优化修改。

接下来这篇文章将会以完成一个完整的shiny app的思路顺序。关于更详细的shiny app的构成要件介绍，还是可以是通过[shiny的速查表](https://www.rstudio.com/resources/cheatsheets/)来更好了解。再往后则是根据两个我做的shiny小品来聊聊shiny app构建的一些共通基本思路。

## UI
### 布局（layout）
布局部分根据需求纲要的确定整个app的大致框架，常见的框架大致如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181216003147535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MzE3MTQ=,size_16,color_FFFFFF,t_70)


flowLayout()，splitLayout()和verticalLayout()适合不同构成要素内容相对均衡的使用场景，并可根据构成要素内容大小多寡具体在三者中选择。

flowRow()和sidebarLayout()适用于构成要素内容差异较大，例如较少的输入要素或需要凸显输出要素等。相对而言，前者适合有一定量的输入要素但输出仍然是需要凸显的情况，后者则是输入要素较少的情况。

### 输入（input）
输入部分是用户在进行交互时操作的对象，是UI界面的直接体现，在UI部分进行定义和设置，并通过「input$\<inputId>」与server部分链接。此外，输入的值都是有反应式的（reactive，最大程度的简化了事件处理代码，从而更专注于应用本身），没有无结果的输入，所以需在server部分写的时候注意每个input都要有对应的反应和输出。

常用的input控件：

![此处仅作展示，具体的代码可查看开始时候提到的速查表](https://img-blog.csdnimg.cn/20181217103545277.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MzE3MTQ=,size_16,color_FFFFFF,t_70)

### 输出（output）

输出部分有两个构件组成，在UI部分呈现的output函数，以及在server部分定义的函数计算的对象。二者是使用时候需要一起考虑。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018121711594153.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MzE3MTQ=,size_16,color_FFFFFF,t_70)


提呈函数|输出函数 |	生成对象
|:----  |:----- | :-----
|renderDataTable|dataTableOutput | DataTable
renderUI	| htmlOutput/ uiOutput |	raw HTML
renderImage	|imageOutput |图片（image）
renderPlot|plotOutput |	图表（plot）
renderTable| tableOutput |	表（table）
renderText|textOutput |文本（text）	
renderPrint	|verbatimTextOutput |	输出值（text，summary()之类的结果）

关于输出部分还需要注意以下几点：
* server部分的呈现函数（render*/* Output）里是存放最终结果的，若从最开始输入需要经过一系列的计算和赋值等过程，就需要借助到以下将会讲到的server的反应部分；

* 每个render* 函数部分的R代码需要用花括号{}收纳；
* 需要将render* 函数的值赋值到output对象，以最开始时候的示例代码为例：

```r
library(shiny)
ui  <- fluidPage( 
plotOutput(outputId = "hist") #UI部分的Output函数，shiny标准的函数一般是也成为render*()函数
)
server  <- function(input, output) { 
output$hist  <- renderPlot({ #UI部分的输出函数的对象内容在server部分进行定义
            hist(rnorm(input$n))
           })
}
```


## 运算（server） 

该部分将重点讨论反应式（reactivity）。如何理解反应式，可以通过这串代码可以体会：

```r
server <- function(input, output) {
output$plot <- renderPlot({
  data <- getSymbols(input$symb, src = "yahoo",
                     from = input$dates[1],
                     to = input$dates[2],
                     auto.assign = FALSE)

  chartSeries(data, theme = chartTheme("white"),
              type = "line", log.scale = input$log, TA = NULL)
})
}
```
如上所示，server部分把所有的计算反应都只放到一个函数renderPlot中，但这也意味着每次运行都在重新获取和计算数据，将会降低app的反应速度和不必要的带宽浪费（尤其是对于shiny server免费用户来说，这种浪费更需要仔细考虑和避免）。更好的方式是这样：

```r
  dataInput <- reactive({
    getSymbols(input$symb, src = "yahoo",
        from = input$dates[1],
        to = input$dates[2],
        auto.assign = FALSE)
  })

  output$plot <- renderPlot({   
    data <- dataInput()
    if (input$adjust) data <- adjust(dataInput())

    chartSeries(data, theme = chartTheme("white"),
        type = "line", log.scale = input$log, TA = NULL)
  })
  
```
本案例来自[shiny入门](http://shiny.rstudio.com/tutorial/written-tutorial/lesson6/)

通过创建对象值（list）dataInput 来隔离两个计算部分。

当然，更重要的是，通过使用不同的反应式，来更多样的控制app的计算和反应过程。

### 直呈式反应

表达式的反应主要是最终导向输出或输出的过程值为目的的反应式。主要包括**reactiveValues()、render\*()和reactive()**。

#### reactiveValues()
输出所设定值函数，与此相对的是在UI部分提到的由用户通过控件产生的输出值：input$\<inputId> 。
```r
library(shiny)
ui <- fluidPage(
textInput("a","")
)
server <-
function(input,output){
rv <- reactiveValues()
rv$number <- 5
}
shinyApp(ui, server)
```
#### render\*()
输出运算结果对象。
```r
library(shiny)
ui <- fluidPage(
textInput("a","")
)
server <-
function(input,output){
output$b <-
renderText({
input$a
})
}
shinyApp(ui, server)
```
#### reactive()
输出运算过程值，作为模块化编程的重要组成。使用运算结果时，需要用函数的可是来调用。具体来说主要有以下三个功能：
* 缓存运算值，减少运算；
* 运算值可被方便多处使用；
* 调试时能够清晰展现问题点

```r
library(shiny)
ui <- fluidPage(
textInput("a",""),
textInput("z", "")
)
server <-
function(input,output){
re <- reactive({
paste(input$a,input$b})
output$b <- renderText({
re()})
}
shinyApp(ui, server)
```

### 控制式反应

控制式反应是在正常的输入-运算-输出之外的反应方式。具体来说包含这三个类型：

#### isolate()
运行代码，但抑制输出结果，返回一个未反应的结果，从而达到避免依赖性（dependency）的目的。理解isolate()，一般可以对比reactive()。reactive()的反馈是实时的、依赖性的，isolate()则是条件性的、非依赖性的。

可在本地运行以下示例app，对比两类反应的结果：
```r
library(shiny)
ui<-
    fluidPage(
    titlePanel("isolate example"),
    fluidRow(
        column(4, wellPanel(
            sliderInput("n", "n (isolated):",
                        min = 10, max = 1000, value = 200, step = 10),          
            textInput("text", "text (not isolated):", "input text"),
            br(),
            actionButton("goButton", "Go!")
        )),
        column(8,
               h4("summary"),
               textOutput("summary")
        )
    )
)

server <- function(input, output) {

    output$summary <- renderText({
        # isolate()一般搭配条件性的触发器使用，其触发器可直接置于其前
        input$goButton
        # 此处的对于str的赋值，类同于reactive，都是实时性的
        str <- paste0('input$text is "', input$text, '"')  
        # isolate()则抑制以下部分的运算进行，从而起到独立性和隔离作用
        isolate({
            str <- paste0(str, ', and input$n is ')
            paste0(str, isolate(input$n))
        })
    })
    
}
shinyApp(ui, server)
```
参考自：[isolate-demo](https://shiny.rstudio.com/gallery/isolate-demo.html)
#### reactive()、observe()、observeEvent()和eventReactive()对比
observeEvent()和eventReactive()两类都属于控制式反应，与直呈式反应的reactive()和observe()的最直接差异在于：前者是延迟性的反应，其输入值（input value）依赖是部分，通过一定的方式（session）触发；后者则是即时计算的，全局性的依赖于输入值。

关于两个大类中的两个小类则在reactive类的是输出对象值的，observe类的是直接作为环境值输出的。

可在本地运行以下示例app，具体对比四类反应的结果：
```r
library(shiny)

ui<-
    fluidPage(
        fluidRow(
            column(3,
                   h2("Reactive Test"),
                   textInput("Test_R","Test_R"),
                   textInput("Test_R2","Test_R2"),
                   textInput("Test_R3","Test_R3"),
                   tableOutput("React_Out")
            ),
            column(3,
                   h2("Observe Test"),
                   textInput("Test","Test"),
                   textInput("Test2","Test2"),
                   textInput("Test3","Test3"),
                   tableOutput("Observe_Out")
            ),
            column(3,
                   h2("ObserveEvent Test"),
                   textInput("Test_OE","Test_OE"),
                   textInput("Test_OE2","Test_OE2"),
                   textInput("Test_OE3","Test_OE3"),
                   tableOutput("Observe_Out_E"),
                   actionButton("Go","Test")
            ),
            column(3,
                   h2("eventReactive Test"),
                   textInput("Test_eR1","Test_eR"),
                   textInput("Test_eR2","Test_eR2"),
                   textInput("Test_eR3","Test_eR3"),
                   tableOutput("eventReac_out"),
                   actionButton("Go_event","Test")
            )
        )
    )

server<-function(input,output,session){
    
    # reactive()和observe()在最终呈现上没有区别，都是随着输出值的实时更新计算输出值的；
    # 二者的区别在于前者输出的Reactive_Var是全局可用的，而后者输出的df则是环境局限的
    Reactive_Var<-reactive({c(input$Test_R, input$Test_R2, input$Test_R3)})
    output$React_Out<-renderTable({
        Reactive_Var()
    })
    
    observe({
        A<-input$Test
        B<-input$Test2
        C<-input$Test3
        df<-c(A,B,C)
        output$Observe_Out<-renderTable({df})
    })
    
    # observeEvent()和eventReactive()同样在最终呈现上没有区别，但在环境调用上存在不同。
    observeEvent(input$Go, {
        A<-input$Test_OE
        B<-input$Test_OE2
        C<-input$Test_OE3
        df<-c(A,B,C)
        output$Observe_Out_E<-renderTable({df})
    })
    
    eventReactive_Var <- eventReactive(input$Go_event, {
        c(input$Test_eR1, input$Test_eR2, input$Test_eR3)})
    output$eventReac_out<-renderTable(eventReactive_Var())
    
}
shinyApp(ui, server)
```
本案例参考自：[Advantages of reactive vs. observe vs. observeEvent](https://stackoverflow.com/questions/53016404/advantages-of-reactive-vs-observe-vs-observeevent)

## 案例
接下来以我做的一个案例为例，来说一下大致的思路：
### 北京地铁月度支出模型
做这个shiny app的初衷是看到[这篇文章](https://www.cnblogs.com/jkisjk/p/4158531.html)，里面关于如何在考虑优惠政策的前提下，计算每月在地铁上的花费。为了更直观的了解地铁花费变化情况。

这个app实现前考虑几个要素：

* 无需输入数据，数据通过函数产生；
* 涉及到的交互：条件选择按钮，文本输入按钮；
* 输出形式，可交互式的图表（本案选择plotly实现）；
* 涉及的的数据字段有三个，通过控件和二维图表进行变化

预览如下，具体可在本地运行查看：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181218150206985.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MzE3MTQ=,size_16,color_FFFFFF,t_70)

```r
library(shiny)
library(ggplot2)
library(plotly)
library(markdown)

ui <- fluidPage(
    # shiny可以调用HTML5静态元素来丰富appUI表现
    tags$style("label{font-family: TT Times New Roman}"),
    tags$style("body{font-family:TT Times New Roman}"),
    titlePanel(HTML("北京地铁月度支出模型 <br/>Beijing Subway monthly Fare Model")),  
    # 该app里的UI元素不复杂，一个条件控件，一个文本输入控件
    fluidRow(
        column(4,radioButtons("radio", label = h4(HTML("X轴选择 <br/> Select X Variable")),
                              choiceNames = c("以天数看花费 \n days as X variable",
                                              "以单日费用看花费 \n day fare as X variable"),
                              choiceValues = c("dayFare","days"),
                              selected = "days")),
        column(5,uiOutput("Input"))),
    # 以及最终的结果呈现，同时，最终结果呈现也可进一步在呈现过程中进行定制化
    plotlyOutput("distPlot", width=800,height = 400)
)

server <- function(input, output) {
    # 生成数据的函数并不需要每次都进行运算，所以通过isolate()进行隔离，从而减少依赖和运算量
    isolate({
        feeInMonth <- function(dayFare, days){
            fee = dayFare * days
            if(fee > 662.5){                                        #662.5 = 100 + 50/0.8 + 250/0.5
                fee = (fee -262.5)} else if(fee > 162.5 & fee <= 662.5){ #162.5 = 100 + 50/0.8   
                    fee = fee/2+68.75 } else if(fee > 100 & fee <= 162.5){#(fee-162.5)/2+150
                        fee = fee*0.8+20 } else { return(fee)}           #(fee-100)*0.8+100
            return(fee)  
        } 
        g <- Vectorize(feeInMonth)
    }) 
    # 通过条件选择呈现不同的按钮
    output$Input <- renderUI({
        if(input$radio == "days"){
            numericInput("Input", label = h4(HTML('每月使用日数<br/> monthly work days')), 
                         value = 22, min = 1, max = 31)
            
        }else{
            numericInput("Input", label = h4(HTML('平均每日花费<br/> average each day fare')), 
                         value = 10, min = 3, max = 50)
        }})
    
    # 最终生成结果。此处用plotly嵌套ggplot的对象值，可以说将R的特点最大程度的发挥，对于熟悉R的来说，最方便不过
    output$distPlot <- renderPlotly(
        {
            if(input$radio == "dayFare"){
                p <- ggplot(data.frame(dayFare = c(3,50),days = c(0,31)), 
                            aes(x = days)) +
                    stat_function(fun = g,args = c(dayFare = input$Input)) + 
                    theme(axis.line = element_line(colour = "darkblue", size = 1.5, linetype = "solid"))+ 
                    labs(x = HTML("使用日数\n using days"), y = HTML("费用\ fare"))
            }
            if(input$radio == "days"){
                p <- ggplot(data.frame(dayFare = c(3,50),days = c(0,31)), 
                            aes(x = dayFare)) +
                    stat_function(fun = g,args = c(days = input$Input)) + 
                    theme(axis.line = element_line(colour = "darkblue",size = 1.5, linetype = "solid"))+
                    labs(x = HTML("平均每日花费\n average each day fare"), y = HTML("费用\ fare"))
            }
            gg <- plotly_build(p) %>%  style(line = list(color = 'lightblue',width = 3))            
        })  
}

shinyApp(ui = ui, server = server,options = list(height = 900))
```





