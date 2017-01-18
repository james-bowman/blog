+++
categories = ["DevOps", "Development"]
date = "2016-02-09T07:49:02Z"
description = "Examination of the pros and cons of standardization across the enterprise"
draft = false
tags = ["devops", "development", "enterprise architecture", "architecture", "standards", "platform", "conways law"]
title = "Standardisation in the Enterprise"
author = "James Bowman"
image = "/post/standardisation_l_crop.jpg"
imageURL = "https://www.flickr.com/photos/togawanderings/14212266277/in/album-72157645145946024/"
imageTitle = "Examination"
imageCreator = "Thomas Galvez"
imageLicenceURL = "https://creativecommons.org/licenses/by/2.0/"
imageLicenceName = "Creative Commons Attribution-Noncommercial license"

+++

In enterprises there is often a strong desire to standardise.  The reasoning is simple: if we are all doing things the same way, using the same technology, then we can simplify our operations, benefit from economies of scale and make our people more fungible.  So by extension, not standardising means duplicated effort, resources and expenditure.  But are things really this clear cut?

Perhaps we should begin by thinking about the meaning of the word standardisation and understanding the alternatives.  [Wikipedia defines standardisation](https://en.wikipedia.org/wiki/Standardization) as:

>Standardization or standardisation is the process of implementing and developing technical standards. Standardization can help to maximize compatibility, interoperability, safety, repeatability, or quality. It can also facilitate commoditization of formerly custom processes. 

As an interesting aside, there is no mention of _uniformity_ in this definition even though the two terms are frequently used synonomously in many enterprises.  Standardisation is often interpreted as application standardisation - normalising and consolidating to uniform, and often centralised, applications and tools.  However, it is important to note there are other types of standards beyond applications - more on these later.

Most Thesaurus' would say the opposite of the word standardisation is _deviation_ which conjours up all sorts of negative conotations.  I would argue that more accurate words to describe the opposite of standardisation, in this context, are _diversification_ and _customisation_.  And, unlike _deviation_, it turns out there are many benefits to diversification and customisation:

- **Lower risk** - reduced exposure to a single key technology, application or supplier
- **Increased responsiveness and agility** - greater autonomy with fewer dependencies and lower impact of change
- **Potentially increased productivity and/or revenue** - tailoring of experience, applications, practices and processes to best suit territories, teams and customers involved

So it seems that both approaches (standardisation and customisation/diversification) have their own, albeit different, sets of benefits and drivers.  Standardisation is often bourne out of attempts to optimise the bottom line (costs) where as diversification and customisation tend to come about when optimising the top line (revenue).  

Of course in reality, things are much more complicated.  Trying to reduce costs through standardisation often simply pushes the costs elsewhere.  For example, consider an organisation comprising 5 separate business units.  Each business unit does similar things but is discreet, perhaps because they are based in different territories or perhaps because they were formerly part of other companies that were acquired.  Each business unit have their own application(s) supporting their own processes, product sets and data structures.  As part of any attempt to consolidate the 5 software applications into one standardised application the organisation must either:

1. Force all business units to adopt the same uniform, and potentially sub-optimal, business processes, product sets and data structures that will be, by definition, a compromise.

	___or___

2. Deal with the complexity and costs of supporting the 5 divergent business processes, product sets and data structures within a single application.

Neither of these options is particular appealing and may ironically cause an increase in cost rather than the desired reduction.  Furthermore, any changes to the application for a particular business unit must now be funneled along with all the changes for the 4 other business units and their impact considered for all 5 business units.  This will result in higher costs and significantly increased lead times for changes.  

This phenomonon is often explained using an adage called [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law).  It states that:

>Any organisation that designs a system (defined more broadly here than just information systems) will inevitably produce a design whose structure is a copy of the organization's communication structure.

The adage illustrates the importance of considering the organisational structure and business processes alongside software applications.  If we attempt to consolidate systems without considering the teams using them we can create misalignment between the organisational structure and systems and incur significant costs as a result.  Clearly, in our increasingly competitive market environments, where businesses are trying to accelerate the development and rollout of new products and services, such costs and delays could lead to missed opportunities. 

## Platforms - the Best of Both Worlds?

Applications have predefined business logic and data structures that narrow their scope and restrict their ability to support divergent business processes and product sets.  In contrast, platforms focus on the truly common, commodity capabilities and separate out the variable logic and data structures, allowing them to diverge.  Platforms strike a balance between standardising and diversifying/customising by splitting the problem domain into that which is truly common (the platform) and that which should be allowed to diverge (the customised items on top of the platform).  Interestingly, platforms are usually driven out of a desire to accelerate product development rather than control costs.

Platforms essentially represent a set of standards or constraints that all the things supported by the platform must conform to.  In return for conforming to these constraints, they are able to leverage all the facilities and capabilities offered by the supporting platform.  In principle, platforms could be built to support anything providing it conforms to the constraints imposed by the platform.  This could include applications, product sets or even business processes.  

Platforms are typically successful when they meet the following 3 criteria:

1. The platform is tightly focused and completely generalised with any variability pushed out into the items it supports.  
2. There is a clear and explicit contract between the platform and the items it supports clearly defining the constraints/standards they must conform to.
3. The platform is mostly self-service, requiring minimal centralised administration or configuration.  This avoids hand-offs and bottlenecks with centralised teams.

## Other Types of Standards

Earlier on, I mentioned that we would later be looking at other types of standard beyond application standardisation.  If we think back to the definition earlier in the article we see that many types of standards could help enterprises including (but not limited to):

- Standards that help us treat things in a consistent way allowing us to build tooling, platforms (see above) and scale operationally (e.g. deployment, hosting, monitoring, etc.)
- Standards that help maximise compatibility and interoperability between software components, business units, etc.
- Standards to promote safety and to keep our assets secure
- Standards to ensure a high level of quality to limit our risk exposure and additional expenses
- Standards that promote consistent user experience to optimise usability and profit

Typically these types of standards are defined across the organisation and can help drive emergent behaviours.  These types of standards explicitly define the small set of things that must be consistent (to support tooling, interoperability, etc.) and leave everything else to vary, allowing teams autonomy and freedom to innovate.

## In Conclusion

There are benefits to both standardising and diversifying/customising and rather than adopting a one size fits all approach, consider each application individually.  However, be wary of approaching standardisation as a means of cutting costs - you may end up simply pushing the costs elsewhere.  Where standardising is desired, you might wish to consider building platforms to support diversity and customisation whilst still reducing time to market by leveraging a focused, common platform.  

Consider rolling out lightweight standards across the enterprise to promote consistency in the areas that consistency is important (e.g. application integration, security, user experience, etc.) helping to accelerate product development but be careful not to restrict teams' autonomy or ability to innovate.

Do any of the points in this article ring true with you?  Have you observed similar things in the companies you have worked for?  If so, please comment and share your experiences.  

[1]: https://creativecommons.org/licenses/by/2.0/
