***
##Project

###Project Name

Refactor Rails' Cookie Implementation and Improve Signing with Expiry and Purpose

###Project Description

Current Rails Cookie System does not have a mechanism to expire a cookie, and purpose of a cookie is not defined. Apparently, cookies are not very secure. This project will make the expiry of cookies possible on the server side and also add purpose field for cookies, so that a cookie does only its purpose and can not be used for something it isn't meant for. To make this integration with the current structure, I will also refactor the cookie internals. Upgrade paths will also be taken care of, so that cookies are readable even after a user upgrades his application.

###Why did you choose this idea?

Amongst all the ideas, this idea matched best with my skills set. I found it really interesting as I researched on it. Every challenge that I faced during the research made me want to do it even more. I also believe that this project will make it easy to generate and store secure cookies.  

###Please describe an outline project architecture or an approach to it

#####Problem
Expiration of cookies is enforced by the browser, so it is possible for a user to change their expiry time. However, we may want to enforce expiration from server side to some degree. Another problem is that a cookie may be misused in place of another cookie having a different purpose. Therefore, a new parameter needs to be introduced for the purpose of the cookie. Below is a link of the application I have made in Rails demonstrating this problem:

[github link](https://github.com/sbhatore/Demo_app_1)

We need to refactor the cookie implementation in order achieve the above integration. However, the code is structured in such a way that refactoring does not seem to be an easy task.

Another potential problem is that of Legacy cookies. Cookie format must be updated as user accesses a cookie after upgrading his application. Since the current implementation for a cookie has only its value signed in it, the expiry and purpose we will assign them in the new format needs to be determined. Applications written from scratch in Rails 5 should refuse to honor any legacy cookies for maximum security. Also, we need to stop honoring legacy cookies after some time into the upgrade of the application. But, what amount of time is acceptable for this?

A problem exists for the browser session cookies. We want to sign the expiry inside the cookie so that expiry can be enforced on server side. Now, what expiry should we sign for browser session cookies which expire after closing the browser?

#####Approach
I will add a purpose parameter for cookies. Then, expiry and purpose of a cookie can be signed or encrypted as part of cookie to solve the problem. This means we will send an encoded Hash containing (value, expiry, purpose) to set cookie as opposed to the current implementation that sends only the encoded "value of the cookie". Thus, we can raise an error if server receives an expired cookie from the browser. Similarly, purpose can be checked too.

Using JSON Web Tokens(JWTs) instead of current representation will help simplify the refactoring problem. A separate class for JWT claims can be made. It will accept payload, and options which will contain purpose and expiry of the cookie and check if the cookies are legit. I will also make a new Abstract class which would be inherited by `SignedCookieJar` and `EncryptedCookieJar` class. This class will take care of serializing headers and claims, checking tampering, etc. Following is a code sample on  how I will like to proceed :

[github link](https://github.com/rails/rails/commit/44c85d059014a4e8fb8a3a32f83e1eb1ec404db8)

`MessageVerifer`'s methods, `generate` and `verify` need to be changed to align with the new format of JWT signed cookies. 

Currently, a cookie can be checked of its legacy (i.e. it is of old version or new version) using environment variables secret_token and secret_key_base. A similar method can be applied for this project too. For the new version, we can have another name for the application's secret token, say secret_token_base. This will be used just like secret_key_base is used now. So, using secret_token/secret_key_base and the new secret_token_base, we can deal with all the upgrade paths for legacy cookies.

I am still researching to find out a better way to deal with the last problem. A version number can be signed inside the JWT header. So, when in future, a user upgrades from Rails 5 to above version, we will have an inbuilt mechanism to determine if the cookie is a legacy cookie or not. 
###Give us details about the milestones for this project
I will use Agile Development Methodology for this project. Given below is a rough timeline for my project, subject to change by mentor intervention:

* **Community Bonding Period( till 24 May)** : Get acquainted with and get more comfortable with the Ruby on Rails community. I will go through the source code of cookie implementation more thoroughly and polish my approach for this project. Since I am new to the community, I will also solve some existing bugs for it, preferably related to my project, so that I become more comfortable with the process of contributing to Rails and learn from the feedback of others. 
* **Week 1** 
 * **(25 May - 27 May)** : Replace the secret_key_base with a new name(say secret_token_base) everywhere. So, we will be basically using secret_token_base in the new version to sign the cookies. So, in this period, I will make changes in Rails to adapt to the new secret name, secret_token_base. I will also make sure that secret_key_base is there just like secret_token, so that it can be used for upgrading cookies. Write tests.
 * **(28 May - 31 May)** : Introduce a new class 'Claims' in `ActionDispatch::Cookies`, and write methods for it. It will accept payload, and options which will contain purpose and expiry of the cookie and check if the cookies are legit. I will also make a new Abstract class which would be inherited by `SignedCookieJar` and `EncryptedCookieJar` class. This class will take care of serializing headers and claims, checking tampering, etc. Write tests for it.
* **Week 2(1 June - 7 June)** : This period will be focused on conversion of the current `ActiveSupport::MessageVerifier`'s conventions for signing cookies into JWT format of signing cookies. Claims containing value, expiry and purpose will be signed into the cookie along with the header after Base64 encoding of them. Methods of `MessageVerifier`, `generate` and `verify` will be changed to align with the new JWT format. Similar process will be applied for encryption of cookies.
* **Week 3(8 June - 14 June)** : Complete the basic implementation for the legacy cookies, so that old cookies on access are converted to new format. Both the upgrade paths need to be taken care of. First is when user upgrades from 3.x to 5 and another is from 4.x to 5. secret_token in 3.x and secret_key base in 4.x will be used to migrate the cookies to the new format of cookies which uses secret_token_base. Write Tests.
* **Week 4(15 June - 21 June)** : Find and fix any bugs in the code, and improve the implementation done so far. Polish the code written till now. Improve testing by writing more tests for the overall implementation.
* **Week 5**
  *  **(22 June - 25 June)** : Write documentation for the work done so far.
  *  **(26 June - 28 June)** : Evaluate the progress made so far with the mentor and make any changes if needed.
* **Week 6**
  * **(29 June - 2 July)** : Submit mid-term evaluations to Google.
  * **3 July** : mid-term evaluations deadline.
 
 * **
Till now, we have improved signing and encrypting of cookies, and also taken basic care of the upgrade path. Now the focus of the project will be to improve and add to the implementations done so far.

  * **
* **Week 7(6 July - 12 July)** : Research more on the prevailing problems and come up with more ideas through discussions with the mentor.
  
* **Week 8(13 July - 19 July)** : Deal with the problems of legacy signed cookies. Assign proper expiry and purpose to them in the new format. Write tests.
* **Week 9** :
* * **(20 July - 22 July)** : Refactor the areas of cookie implementation.
* * **(23 July - 26 July)** : Deal with other problems of the new cookie format implementation like what expiry to sign for a browser session cookie. Find and fix bugs in the code.
* **Week 10**
  * **(27 July - 30 July)** : Write tests and do the testing. Improve the quality of the code written.
  * **(31 July - 2 August)** : Add to the documentation for the work done.
* **Week 11(3 August - 9 August)** :  Buffer week to accommodate for any emergencies, unexpected delays, etc. Evaluate the progress made so far with the mentor and make any changes if needed. 
* **Week 12(10 August - 16 August)** : Polish the code, tie up loose ends.
* **Week 13**
  * **17 August** : Suggested Pencils down date. Scrub the code, write tests, improve documentation. 
  * **21 August** : Firm 'pencils down' date. Final submission to Google.
 
 * **
  
**Note** : If the proposal goes faster than expected, I will look into other low hanging security concerns of Rails, e.g, verbose server headers concern.

###Why will your proposal benefit Ruby on Rails?

Today, Cookies are an indispensable part of any website. A web application might require use of cookies which contain sensitive data. Therefore, it becomes very necessary to store this data securely. Through my implementation, Developers will be able to make much more secure cookies than before very easily.

Merged PR(s)
1. https://github.com/rails/rails/pull/19852 
2. https://github.com/rails/rails/pull/19823

** *

##Open Source
###Please describe any previous Open Source development experience

I am new to Open Source development. Ruby on Rails is the first Open Source community I've joined. 
###Why are you interested in Open Source?

I gained a lot of interest in developing web based applications after I developed some of them as a part of my academic projects. Open Source has opened the doors for me to interact with Web and Software Developers all around the world. I have been learning a lot from other members of the community ever since I joined Ruby on Rails. Open Source has provided me the opportunity to work with other amazing people and to contribute back to the product I love using.
 
 ** *
 
##Availability
###How long will the project take? When can you begin?

The project would take roughly 3 months to complete. I can start the project from 8th May 2015.
###How much time do you expect to dedicate to this project? (weekly)

I expect to devote **40-45 hours** per week on this project. 
###Where will you be based during the summer?

I will be based in **Hyderabad, India** this summer.
###What timezone will you be working in and what hours do you plan to work? (so we can best match mentors)

I will be working in [IST](http://en.wikipedia.org/wiki/Indian_Standard_Time)(UTC + 5:30). I mostly work around midnight(say 22:00 - 6:00). However, I am flexible with my schedule and can work anytime of the day. Though not mentioned explicitly, I plan to take Sundays off from my work.
###Do you have any commitments for the summer? (holidays/work/summer courses)
My vacations are from 8th May - 3rd August, during which I don't have any other commitments.
###Have you ever participated in a previous GSoC? If yes, describe your project.

No.
###Have you applied for any other 2014 Summer of Code projects? If yes, which ones?

No.
###Why did you apply for the Google Summer of Code ?


Participating in GSoC would be a wonderful opportunity for me to work on an Open Source Project. Working on this project full time would help me hone my developer skills and acquire new skills. The experience I'll gain from this would be immense, which I shall use in my future contributions to Open Source and Projects.         
###Why did you choose Ruby on Rails as a mentoring organization?

I used Ruby on Rails framework to develop an application as one of my academic projects, and I was immediately attracted to it. So, when I was first drawn to Open Source I chose Ruby on Rails. As I went through the source code, I began to appreciate the beauty of RoR. A small piece of code in RoR can work wonders. Its vast community which is constantly active is overwhelming. That's why I would love to contribute to Ruby on Rails.
###Why do you want to participate and why should Ruby on Rails choose you?

I am currently in the 4th semester of my ongoing degree. In these two years, I’ve completed the following technical courses (in chronological order):

Computer Programming, Digital Logic and Processors, Mathematics 1, ITWS 1, Data Structures, Computer System Organization, Basic Electronic Circuits, Mathematics 2, ITWS 2, Algorithms, Operating Systems, Introduction to Databases, **Structured System Analysis and Design(SSAD)**, Mathematics 3, Science I.

These courses are still going on:- Graphics, Engineering Systems, Formal Methods, Introduction to Artificial Intelligence, Digital Signal Analysis and Applications, **Computer Networks**

These courses give an insight into my technical knowledge and skills. Below are some projects I've worked on showing my experience-:

* **Surrogate City Finder Tool** An application which uses RoR framework and Google Maps API to find cities whose weather conditions match with that of a particular city.  The code is written mostly in JavaScript as it revolves around plotting cities using Google Maps API. I developed it as a project under SSAD course. The project follows various concepts of Software Design such as requirements specification, testing, refactoring, documentation, etc. ([github link](https://github.com/sbhatore/SSAD46))
* **Skynote** An application which uses Web2py framework to maintain personal notes for a user. It was another of my School Projects. ([github link](https://github.com/sbhatore/ITWS2))

 I've developed a couple of more applications and also a 2-D game using OpenGL.  

On the technical part, I am familiar with refactoring and quite experienced in developing Rails applications. I've also developed a good understanding of cookies and their current implementation in Ruby on Rails as a result of extensive research I've been doing on this project for the past 3 weeks. I've become really familiar with the code structure of the cookie implementation.

I am new to Open Source, and this is why it's all a lot more exciting. The fact that my code would be used by so many people is a great impetus for me to work on it all that much more. 

I study in a college where academic workload is remarkably high on its students. So, it becomes essential for us to master the art of time-management. Notably, I play football regularly in the evenings and I'm also part of my college team. I've learned to maintain a balance between my academics and other interests. This skill of mine would be useful in completing the project punctually and successfully. If need be, I will have no problem working longer hours too as I am really excited to do this project. I'm also quite fluent in English, so you will have no problem communicating with me.

I want to contribute to Ruby on Rails in a long run even if I don't get selected. But if I get to do this project, my motivation, skills, and experience as a contributor will boost-up to a different level. Given this opportunity, I will make the most of it and make the open source world a better place through my contribution. 

