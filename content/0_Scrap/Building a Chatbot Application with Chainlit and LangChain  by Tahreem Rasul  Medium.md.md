---
created: 2024-07-23
url: https://medium.com/@tahreemrasul/building-a-chatbot-application-with-chainlit-and-langchain-3e86da0099a6
tags: 
  - Clipping
---

# Building a Chatbot Application with Chainlit and LangChain | by Tahreem Rasul | Medium

> ## Excerpt
> In this article, we will develop an application interface for our custom chatbot, Scoopsie, using Chainlit, a framework that simplifies the creation of chatbot applications with a ChatGPT-like…

---
[

![Tahreem Rasul](https://miro.medium.com/v2/resize:fill:88:88/1*whixGo6nrkNkKhlYSE9UUw.jpeg)



](https://medium.com/@tahreemrasul?source=post_page-----3e86da0099a6--------------------------------)

In this article, we will develop an application interface for our custom chatbot, **Scoopsie**, using [Chainlit](https://docs.chainlit.io/get-started/overview), a framework that simplifies the creation of chatbot applications with a ChatGPT-like interface. This is a continuation of the series where I previously explored building a custom chatbot tailored to specific use-cases using [LangChain](https://www.langchain.com/) and [OpenAI](https://openai.com/). If you haven’t read the first article, I recommend starting there for a detailed step-by-step guide.

![](https://miro.medium.com/v2/resize:fit:700/1*zyuHtG28zRMwIa1mycgYXw.png)

Application Frontend with Chainlit

Here is the link to the first article in the series:

By the end of this tutorial, you will learn how to build the application interface for your custom chatbot, similar to this:

![](https://miro.medium.com/v2/resize:fit:700/1*YRp7gT3rySVA3JWHwd_rFQ.gif)

Preview of the Completed Chatbot Application Interface using Chainlit

In the [previous article](https://medium.com/@tahreemrasul/how-to-build-your-own-chatbot-with-langchain-and-openai-f092822b6ba6), we laid the foundations for creating **Scoopsie**, an ice-cream assistant chatbot built to answer ice-cream related queries, using LangChain and OpenAI (see [GitHub](https://github.com/tahreemrasul/simple_chatbot_langchain/tree/main) for code). Here’s how our `chatbot.py` looks from the last tutorial:

```
<span id="896f" data-selectable-paragraph=""><span>from</span> langchain_openai <span>import</span> OpenAI<br><span>from</span> langchain.chains <span>import</span> LLMChain<br><span>from</span> prompts <span>import</span> ice_cream_assistant_prompt_template<br><br><span>from</span> dotenv <span>import</span> load_dotenv<br><br>load_dotenv()<br><br>llm = OpenAI(model=<span>'gpt-3.5-turbo-instruct'</span>,<br>             temperature=<span>0</span>)<br>llm_chain = LLMChain(llm=llm, prompt=ice_cream_assistant_prompt_template)<br><br><span>def</span> <span>query_llm</span>(<span>question</span>):<br>    <span>print</span>(llm_chain.invoke({<span>'question'</span>: question})[<span>'text'</span>])<br><br><span>if</span> __name__ == <span>'__main__'</span>:<br>    query_llm(<span>"Who are you?"</span>)</span>
```

Currently, **Scoopsie** lacks the ability to remember past interactions and doesn’t have a user interface. In this article, we will be focusing on equipping our chatbot with memory for more contextual conversations and creating a web application interface using Chainlit.

## Environment Setup

If you haven’t set up a `conda` environment for the project yet, you can go ahead and create one. Remember that Chainlit requires `python≥3.8`. You can skip this step if you previously created your environment.

```
<span id="baca" data-selectable-paragraph="">conda create --name chatbot_langchain python=3.10</span>
```

Activate your environment with:

```
<span id="f457" data-selectable-paragraph="">conda activate chatbot_langchain</span>
```

To install Chainlit along with any other dependencies, run:

```
<span id="2a5d" data-selectable-paragraph="">pip install -r requirements.txt</span>
```

## Understanding Chainlit

Chainlit is an open-source Python library designed to streamline the creation of chatbot applications ready for production. It focuses on managing user sessions and the events within each session, like message exchanges and user queries. In Chainlit, each time a user connects to the application, a new session is initiated. This session comprises of a series of events managed through the library’s event-driven decorators. These decorators act as triggers to carry out specific actions based on user interactions.

The Chainlit application has decorators for several events (chat start, user message, session resume, session stop, etc.). For our chatbot, we’ll concentrate on writing code for two key events: starting a chat session and receiving user messages.

## Initializing the Chat

The `@cl.on_chat_start` decorator is triggered when a new chat session is created. It calls a function that sets up the chat environment, involving initializing our model, creating an LLMChain object, and setting up any necessary variables for operation, including chat history initialization (more on this later).

## Processing Messages

The `@cl.on_message` decorator is used for processing incoming messages from users. It triggers an asynchronous function — suitable for operations that might need to wait for external processes, like model queries. This method utilizes the session’s stored `llm_chain` object, processes the incoming message using the retrieved chain and sends back the response to the application.

## Step-by-Step Implementation

## Step 1:

Previously, our chatbot lacked context for any previous interactions. While this limitation can work in standalone question-answer applications, a conversational application typically requires the chatbot to have some understanding of the previous conversation. To overcome this limitation, we can create a memory object from one of LangChain’s memory modules, and add that to our chatbot code. LangChain offers several memory modules. The simplest is the `ConversationBufferMemory`*,* where we pass previous messages between the user and model in their raw form alongside the current query.

Let’s import the memory module inside the `chatbot.py` file:

```
<span id="534d" data-selectable-paragraph=""><span>from</span> langchain.memory.buffer <span>import</span> ConversationBufferMemory</span>
```

Next, define the memory object to add to the `llm_chain` object:

```
<span id="0282" data-selectable-paragraph="">conversation_memory = ConversationBufferMemory(memory_key=<span>"chat_history"</span>,<br>                                               max_len=<span>50</span>,<br>                                               return_messages=<span>True</span>,<br>                                                   )</span>
```

In the code above, `memory_key` defines what variable the chain will use to store the conversation history in. By default, our chain and prompt expect an input named `history`. We can control this variable by passing in a different value for the parameter.

We also need to add the `chat_history` variable to our prompt template:

```
<span id="78c2" data-selectable-paragraph=""><span>from</span> langchain.prompts <span>import</span> PromptTemplate<br><br>ice_cream_assistant_template = <span>"""<br>You are an ice cream assistant chatbot named "Scoopsie". Your expertise is <br>exclusively in providing information and advice about anything related to <br>ice creams. This includes flavor combinations, ice cream recipes, and general<br>ice cream-related queries. You do not provide information outside of this <br>scope. If a question is not about ice cream, respond with, "I specialize <br>only in ice cream related queries."<br>Chat History: {chat_history}<br>Question: {question}<br>Answer:"""</span><br><br>ice_cream_assistant_prompt_template = PromptTemplate(<br>    input_variables=[<span>"chat_history"</span>, <span>"question"</span>],<br>    template=ice_cream_assistant_template<br>)</span>
```

## Step 2

Let’s now write some code to create our Chainlit application. As we talked in the beginning of the article, we need to write wrapper functions for two Chainlit decorators. Let’s first import the library in our `chatbot.py` file:

```
<span id="14ab" data-selectable-paragraph=""><span>import</span> chainlit <span>as</span> cl</span>
```

The first function will be around the chat initiation decorator: `@cl.on_chat_start`. This function prepares our model, memory object, and `llm_chain` for user interaction. The chain object is passed to the user session, and a name is specified for the session. This name can be used later on to retrieve a specific chain object when a user sends a query.

```
<span id="9521" data-selectable-paragraph=""><span>@cl.on_chat_start</span><br><span>def</span> <span>quey_llm</span>():<br>    llm = OpenAI(model=<span>'gpt-3.5-turbo-instruct'</span>,<br>                 temperature=<span>0</span>)<br>    <br>    conversation_memory = ConversationBufferMemory(memory_key=<span>"chat_history"</span>,<br>                                                   max_len=<span>50</span>,<br>                                                   return_messages=<span>True</span>,<br>                                                   )<br>    llm_chain = LLMChain(llm=llm, <br>                         prompt=ice_cream_assistant_prompt_template,<br>                         memory=conversation_memory)<br>    <br>    cl.user_session.<span>set</span>(<span>"llm_chain"</span>, llm_chain)</span>
```

Next, we will define the message handling function with the `@cl.on_message` decorator. This function is responsible for processing user messages and generating responses:

```
<span id="e3f9" data-selectable-paragraph=""><span>@cl.on_message</span><br><span>async</span> <span>def</span> <span>query_llm</span>(<span>message: cl.Message</span>):<br>    llm_chain = cl.user_session.get(<span>"llm_chain"</span>)<br>    <br>    response = <span>await</span> llm_chain.acall(message.content, <br>                                     callbacks=[<br>                                         cl.AsyncLangchainCallbackHandler()])<br>    <br>    <span>await</span> cl.Message(response[<span>"text"</span>]).send()</span>
```

In the code above, we retrieve the previously stored `llm_chain` object from the user session. This object holds the state and configuration needed to interact with the language model. The `llm_chain.acall` method is then called with the content of the incoming message. This method sends the message to the LLM, incorporating any necessary context from the conversation history, and awaits the response. Once the response from the LLM is received, it’s formatted into a `cl.Message` object and sent back to the user.

## Step 3

With the application code ready, it’s time to launch our chatbot. Open a terminal in your project directory and run the following command:

```
<span id="7bbf" data-selectable-paragraph="">chainlit run chatbot.py -w --port 8000</span>
```

The `-w` flag tells the application to automatically reload when we make some changes to our application code. You can access the chatbot by navigating to [**http://localhost:8000**](http://localhost:8000/) in your web browser.

Upon launching, you will get the default view of the application:

![](https://miro.medium.com/v2/resize:fit:700/1*kkGENsKuGqn8xCsTGBdJMg.png)

Default View of the Chatbot Application Upon Launch

## Step 4

We can make changes to the welcome screen by modifying the `chainlit.md` file at the root of our project. If you do not want a welcome screen, you can leave this file empty. Let's go ahead and add a description relevant to our chatbot.

```
<span id="621b" data-selectable-paragraph=""><span># 🍨 Welcome to Scoopsie! 🍦</span><br><br>Hi there! 👋 I am Scoopsie and I am built to assist you with all <br>your ice-cream related queries. You can begin by asking me anything<br>related to ice-creams in the chatbox below.</span>
```

We can also make changes to the app’s appearance such as changing the theme colors by making changes to the `config.toml` file. You can find this file in the `.chainlit` folder inside the project. For now, I will just go with the defaults. The app also supports toggling between light and dark mode directly from the frontend:

![](https://miro.medium.com/v2/resize:fit:700/1*l8IytTdGuYt0bCDN4IYQCQ.gif)

How to Toggle Between Light and Dark Mode in Your Chainlit Application

## Demo

**Scoopsie’s** application interface is now ready! Here is a demo showcasing the chatbot in action:

![](https://miro.medium.com/v2/resize:fit:700/1*DfkQgUPOdRpJigysTA19qA.gif)

Scoopsie Chatbot Demo: Interactive Ice-Cream Assistant in Action

## Next Steps

Our custom chatbot’s application interface is all set up. In the next tutorial, we will be focusing on integrating an external API with our chatbot, as this can be a useful feature in several enterprise-level applications.

You can find the code for this tutorial in this [GitHub](https://github.com/tahreemrasul/simple_chatbot_langchain/tree/main) repo. The [GitHub checkpoint](https://github.com/tahreemrasul/simple_chatbot_langchain/tree/1427d5d2efac487916e37c7f10183ff600809d0a) for this tutorial will contain all developed code up until this point.

You can [follow along](https://medium.com/@tahreemrasul) as I share working demos, explanations and cool side projects on things in the AI space. Come say hi on [LinkedIn](https://www.linkedin.com/in/tahreem-r-20b7b8218/)! 👋
## 요약 및 주제
-