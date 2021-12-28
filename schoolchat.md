# DIDComm and SSI using Aries Framework JavaScript on Node.js with Aries Mobile Agent React Native
## The goal of the project is to design a messaging application called "School Chat" that allows students who are considering applying to a school to chat with other students who currently attend that school so that they can ask questions and get information that might help them decide if they want to apply.

Organizational Structure

- Multiple schools will participate in this program and all of the participating schools belong to an organization called "The School Group".
- The School Group is the authority for whether or not a school is a member of its organization.
- The School Group is also the authority for which students currently attend which schools in the organization.
- All of the colleges in The School Group are provided with an account that allows them to log in to The School Group administrative website and make changes to the profiles of students that attend their college.

Entities Involved in this example scenario

- **The School Group**<br/>
An organization consisting of a group of schools that all agree to operate under the same policies. The organization can add new schools at any time.
- **Popular College**<br/>
A college that is a member of The School Group. A college can add new students at any time.
- **Alice**<br/>
A student who is considering attending Popular College.
- **Bob**<br/>
A student who currently attends Popular College.
- **Employee**<br/>
An employee of Popular College with the ability to log in to The School Group administrative website and make changes to the profiles of students that attend Popular College.
