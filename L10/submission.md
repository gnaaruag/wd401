### Application context

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a pair is formed the users can share between themselves doodles.
### Exploration of Potential Applications

there are quite a few Gen AI applications that i have explored for this submission i will go over them and elaborate on them in this section

##### 1. AI-Powered Doodle Suggestions:

We can try to implement image generation models and finetune them to our application so that the user can generate doodle messages via textual prompting while still maintaining a doodle-like look and feel

##### 2. GenAI driven doodle enhancement

We could use models like stable diffusion, controlnet-scribble to get user drawn doodles and convert them to hyper-realistic images that can be shared around

A button could be added to integrate this so that it is done as an opt in feature

##### 3. GenAI driven animation generation

We could include a feature which takes the user drawn doodle as an input and generate tiny animations based on that to share with friends.

When this mode is activated we can generate a small animation for this application

#### 4. Natural Language Interaction

A conversational large language model can be integrated in order to textually simulate user navigation through the app

This could be an input bar that appears on clicking a certain keyboard shortcut or accessed from the home page

Example: "Invite Alex to my pair", "Navigate to free draw mode" etc.

##### 5. GenAI doodle suggestions

We can use an LLM and integrate with the application and suggest various prompts as ideas for users to doodle about.

This could be generated on press of an idea button for the user to get inspired

Above are five different use cases/applications for GenAI integration into the application
### Challenges and decisions encountered

In this section we will discuss challenges faced, challenges faced in implementation, a few practical pointers as to whether an integration is necessary and overall if it even makes conceptual sense to integrate GenAI

Below I will describe challenges/roadblocks/caveats for all of the ideas suggested in the above section
##### 1. AI-Powered Doodle Suggestions:

An integration of diffusion models to increase quality of doodles might take away from the sense of uniqueness and personality from the application

From a technical standpoint it is not a difficult task to integrate such a feature, but a consideration to be made is even if we can integrate it should we integrate it
##### 2. GenAI driven doodle enhancement

GenAI doodle enhancement again suffers from the same pitfalls as the doodle suggestions, it might take away a sense of personality between users

Again technically it is easily implementable, we can obviously make this an opt in feature so that it doesn't feel intrusive for a user
##### 3. GenAI driven animation generation

GenAI animation generation can be a particularly interesting application as it adds to an experience that is already being provided and can help retain users

A big challenge is video/animation generation models like flux, sora are very expensive and it will not make sense to integrate into the application as we do not maintain the application as a free service and charge the users no money
##### 4. Natural Language Interaction

This feature can be integrated very easily, and has no major pitfalls or considerations. this can added to the home screen so that it is non obtrusive and users who don't prefer this approach can keep doing their own thing.
##### 5. GenAI doodle suggestions

This feature can be integrated very easily as well, a button can be added near the palette tools which when pressed generates a textual prompt so that a user can use it as inspiration

#### List of Concerns

1. Cost of Integration
2. Loss of originality/personality
3. Intrusiveness
4. Loss of intuitiveness of the UI

#### Decisions

After evaluation of all ideas and comparing them to the list of concerns I have decided to integrate the following features

1. Natural Language Interaction
2. Doodle Suggestions via prompting

The above mentioned features are easy to integrate, do not cost a lot of money, are not intrusive, and doesn't cause a loss in intuitiveness of the UI


### Critical Evaluation of using GenAI w.r.t the web application

The integration of these features can have positive as well as negative impacts on the web application experience

It can have positive impact on the following features
- Greater personalization
- Enhanced interaction
- Targeted improvements to the UX

It also has many of the following negative impacts
- Loss of authenticity
- Complexity
- Intrusiveness in user flow
- Maintenance burden

While GenAI can offer numerous potential benefits, it also has potential to cause issues in the application therefore careful consideration must be put into what features of GenAI to integrate.

Therefore after considering lots of ideas for GenAI integration, I have settled on the two mentioned in the previous section. 

The core application functionality itself doesn't benefit a lot from GenAI integration therefore I have decided to limit the scope of AI in the application to textual UX and inspiration prompts. 

I have evaluated benefits and costs of integration carefully and have decided to go ahead with the aforementioned ideas to implement.

