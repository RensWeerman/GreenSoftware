## Introduction

In this article, we will show our work in making the team chat platform Zulip, more energy efficient and sustainable, using certain strategies for achieving green software. Zulip aims to reduce communication clutter and improve the organization of conversations through its threading model. Unlike other applications, Zulip organizes conversations into topics within channels which makes it easier to follow multiple discussions simultaneously and catch up on missed conversations. Zulip is accessible across multiple platforms with native applications for Linux, Android, Windows, iOS and macOS as well as a web application and through a terminal. We selected Zulip for the project’s size, and the opportunities that seemed to bring. Additionally, Zulip has a backend service mainly written in python, which was important for one of the green software strategies we wanted to explore.

In order to start reducing the energy consumption of Zulip, we perform a lifecycle analysis. We do this by defining our functional unit to use kWh and by defining the boundaries of the system. We focus on Zulip as a running server, rather than looking at its full lifecycle. This means we primarily worked on estimating and improving the energy use of a running Zulip system, along with the interactions made by potential Zulip users. The diagram below shows the energy processing diagram for Zulip based on its documentation.

![Energy Processing Diagram](https://raw.githubusercontent.com/RensWeerman/GreenSoftware/refs/heads/main/EnergyProcessingDiagram.png)

It shows both the structure of the project we are exploring, and shows the estimated energy cost for specific parts of the system.

Using the lifecycle analysis, we looked for areas that could be improved in Zulip and explored these areas using the strategies of green software and formulating a hypothesis. We selected the following areas to improve:

Python version
Presence status
Message fetching

In order to explore these areas, we decided to focus on the strategies of “transmit less”, “less layers and (unused) features”, and “use lean systems”.

Python version
The first area of improvement we will be focusing on is the usage of Python for Zulip’s backend. Although other systems, such as the database system within the backend, do exist, our idea is that we can generally improve the energy consumption of Zulip’s backend by making adaptations to the software used in the first place. Although quite general, the idea for this first area is that Python versions can influence the speed and energy consumption of running Zulip.

This idea is similar to the work done by Grinberg, and really aims to test if their work in comparing Python versions can be replicated when viewing a single project. In order to check whether our hypothesis is correct, we make use of Zulip’s CI/CD or GitHub Actions, by making adaptations in order to run energy benchmarks for different Python versions. At the start of every workflow, we install Python and run the Zulip API tests for each Python version and estimate the energy consumption/emissions using CodeCarbon. We use estimates from CodeCarbon because obtaining measurements by themselves proved challenging, since Zulip’s development environment is virtualized within a Docker container using Vagrant. Because of this, access to specific hardware components is limited. For example, the Linux command perf, which measures energy consumption, does not work, since it does not have the permission to access the RAPL counter. We found the following estimated emissions:


![Different Python Versions](https://raw.githubusercontent.com/RensWeerman/GreenSoftware/refs/heads/main/Emissions%20for%20Different%20Python%20Versions.png)

With these results, we can see that there is a difference between Python versions. Additionally, as Grinberg’s work also reflected, newer versions of Python seem to produce lower emissions, and the estimates we made of the Zulip system reflect this conclusion.

## Presence Status

Another area we looked at for finding improvements in Zulip is the use of a presence system. The presence status in Zulip provides information about the status of users. The purpose of the presence function is that through status indicators it allows users to quickly identify who is active, idle or offline. These status indicators are displayed adjacent to user names in the user list. The presence feature works by tracking whether the Zulip client is open and in focus in the last 140 seconds.

In the documentation for Zulip, we found that the response of the endpoint, which provides presence data, still sends a legacy format that is set to be removed in the future. We would then expect that the removal of this legacy format, which is now additionally sent in each presence update, would result in a measurable reduction in energy use.

After we removed the legacy format, we were able to observe a change in the estimated energy consumption for the Zulip API tests:


![Legacy Presence Measurements](https://raw.githubusercontent.com/RensWeerman/GreenSoftware/refs/heads/main/Presence_Measurements.png)

We were not able to achieve significant reductions across all tests, since most of the differences are zero or close to zero which is a negligible difference. We were able to achieve the most reduction in the single_user_get test. This test involves repeatedly fetching presence for a single user, therefore computing both the modern and the legacy format requires the most energy.

## Message Fetching

Our last area of improvement focuses on the cost for users to fetch messages from the Zulip server. Besides channels, users in Zulip can also communicate in private messaging, through direct messaging (DM). In these chat environments, information about senders and receivers is not necessary, since it is already clear to the end user which DM they are making a request for. Zulip sends this information for each message, but if we were to remove it for DMs, we could have a potential reduction in bandwidth usage between end-users and the Zulip backend.

We make use of the strategies for “transmitting less” and “batching”, in order to see the following reduction of size in a single message:


![Message Fetching Before After](https://raw.githubusercontent.com/RensWeerman/GreenSoftware/refs/heads/main/Length%20of%20JSON%20object%20(characters).png)

Overall, the estimates before and after our reductions for message fetching do support the idea that there are opportunities to reduce bandwidth costs within Zulip. Although large message sizes would make the impact of reducing metadata size less important, these changes would have a positive impact on energy consumption for most messages.

Even so, the size of the Zulip project meant it was difficult for us to completely test the reductions in message metadata within the entire system. Much of Zulip depends on the data included in a message object, and making changes to it would require more time to completely update the Zulip backend.

## Conclusion

Ultimately, improving Zulip turned out to be more difficult than we expected. Although we chose Zulip primarily for its size, hoping to find meaningful changes, this same size made it difficult to grasp the entire project. Despite this, we were able to explore three areas of potential improvement, finding ways to reduce bandwidth usage and exploring how the systems that are used by a project, in this case a Python interpreter, can lead to improvements in energy consumption.

Lastly, the full report can be found [here](https://github.com/RensWeerman/GreenSoftware/blob/main/Green_Software.pdf).
