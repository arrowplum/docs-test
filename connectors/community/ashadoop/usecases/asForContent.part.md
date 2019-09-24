### Content Personalization

eCommerce is about getting the right product in front of the customer at the right time. Cloud-based technologies, predictive behavior analysis, image processing algorithms & Real Time Bidding (RTB) technology are changing the traditional meaning of Content as a Service (CaaS) in the ecommerce space.

Its like having a secret personal shopper, on the web!

Content as a Service has traditionally been discussed in the context of content management for a single channel (or a CaaS provider’s client). CaaS has also been associated with electronic delivery of print media. “Content” is now evolving to include the inventory of items in an e-commerce catalog.

Meanwhile, RTB has created a marketplace for online advertising on web properties - be it display ads or now the next hottest thing - pre-roll video ads.

RTB is geared towards selecting the winning online ad at a web property. However, if we keep the user profile algorithms in place but instead of serving ads, we put in place a mechanism to display specific catalog items in the e-commerce portfolio, we can replace third party ads with e-commerce catalog items. With most of the pieces staying the same, it is now possible to serve targeted e-commerce content.

#### Extending Real Time Bidding to Content as a Service
A Content as a Service provider, with multiple web retail clients and user behavior data across multiple channels leverages and tailors content at each e-commerce site specific to each visitor. The net result is different visitors see different product selections at an e-commerce website.

The advantages are obvious for retailers. Specific product placement in front of a buyer who is at the point of intent in his purchase-making decision is far more lucrative than placing an ad for a subsequent purchase and hoping it ranks high in the attribution analysis… eventually. 

The challenge is getting major retailers to agree on leveraging the user profile data across multiple channels. It is a relatively easier value proposition for smaller retailers who want access to such technology and do not have the technical know-how or the budget to develop their own indigenous technology.

#### Personalizing E-Commerce Content 
Personalized dynamic content creation works in three basic steps as outlined in the figure below: 


<center>
{{#figure "" "Personalizing E-Commerce Content" size="medium" }}
<img src="/docs/connectors/assets/images/PersonalizingContent.png" >
{{/figure}}
</center>

1. **Candidate Products based on Context:** What is right for the context? Products are categorized depending on the Landing or Home Page or Category Tab by keywords, synonyms, brands, manufacturers, etc., and decompose queries into attributes and products. 
2. **Personalize:** What is right for the consumer? Connect users and products algorithmically based on context such as past search query, past behavior yielding personalized content.
3. **Business Optimization:** What is right for business? Products to be displayed are further refined based on available inventory and margins. If retailer chooses to opt for dynamic pricing, it is influenced by competition and inventory.

#### The Technology Stack
For Content as a Service with algorithms, the tech stack is a variant of the RTB stack. The data collection, crawlers and the like, are flumed into the Hadoop ecosystem. The algorithms use an in-memory database such as Aerospike for extremely fast real time read/write of Key-Value data. 


<center>
{{#figure "" "Content Personalization as a Service" size="large" }}
<img src="/docs/connectors/assets/images/ContentPersonalizationAAS.png" >
{{/figure}}
</center>


The latency challenge is just as demanding as RTB if not more stringent and the stakes are higher. If the customized content takes any longer than 100ms to display, the client will quite likely lose the visitor, and hence a direct sale opportunity. However, if successfully executed, it has a higher probability of adding to the bottom line.

The algorithms work in real time, adjusting content based on user action. If the customer places a red polka dot dress in the cart, complimentary accessories are displayed alongside. A couple of adjacent choices are displayed as well to make sure the algorithm is not trapping the visitor into unwanted choices. If mom is shopping for her teenage daughter’s graduation gift, the algorithm must recognize it and adjust content accordingly, in real-time. 

