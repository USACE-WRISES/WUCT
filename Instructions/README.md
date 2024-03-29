---
title: "Watershed-Scale Upstream Connectivity Toolkit (WUCT)"
author: "S. Kyle McKay"
date: "September 19, 2018"
output: word_document
---
```{r options, include=FALSE}
knitr::opts_chunk$set(warning=FALSE) 
```


## **Overview**   

This model document describes a procedure for quantifying benefits associated with removal of organism movement barriers within a watershed (e.g., dam removal, culvert repair, fish ladder installation) or impacts of barrier addition (e.g., dam construction, weir installation). The model focuses on upstream movement of migratory organisms such as fish and is intended for application at the watershed-scale.The algorithm is based on four primary components: habitat quantity upstream of a dam, habitat quality upstream of a dam, the passability of a structure for a given organism, and the shape/topology of the watershed. This algorithm combines these data to estimate quality-weighted, accessible habitat at the watershed scale. This model documentation is intented for the U.S. Army Corps of Engineers (USACE) model certification process and details the development, technical details, and application of this adaptable modeling framework.  

*Author Notes*  

* Corresponding Author: Kyle McKay, U.S. Army Engineer Research and Development Center (ERDC), Environmental Laboratory (EL), New York, New York, Office: 917-790-8717, Cell: 601-415-7160, kyle.mckay@usace.army.mil.  
* This model documentation was developed in the R Statistical Software using Rmarkdown, which integrates reporting and documentation into a single location.  All code is displayed in-line with text to facilitate review.  
* This report documents ecological model development and application for the purposes of the U.S. Army Corps of Engineers (USACE) Model Certification Program.  The model framework will be applied in the context of the Hudson-Raritan Estuary (HRE) and Hudson River Habitat Restoration (HRHR) projects led by USACE New York District.  
* This toolkit was submitted as a planning model to the Ecosystem Restoration Planning Center of Expertise (ECO-PCX) in July 2018 and reviewed by Michael Dougherty (USACE Rock Island District) and Clayton Ridenour (Omaha District). Review comments and responses are included in Appendix C of this document. Final approval of this model for national certification was given in October 2018.  
* Prior versions of this toolkit included input from Jock Conyngham, Craig Fischenich, John Schramski, Molly Reif, Diana Kohtio, Tom Prebyl, and other co-authors.


## **Background**   

Water resources and transportation infrastructure such as dams and culverts provide countless socio-economic benefits; however, this infrastructure can also disconnect the movement of organisms, sediment, and water through river ecosystems (Pringle 2001). Trade-offs associated with these competing costs and benefits occur globally, with applications in barrier addition (e.g., dam and road construction), reengineering (e.g., culvert repair, fish ladder installation), and removal (e.g., dam removal and aging infrastructure). Nationwide, millions of barriers inhibit the movement of aquatic organisms, and barrier removal and repair have emerged as key techniques for river restoration.  As of 2016, over 1,200 dams have been removed nationwide (Bellmore et al. 2016) as well as countless road-stream crossings repaired to facilitate movement (Januchowski-Hartley et al. 2013). 

Restoration practitioners require tools for informing barrier removal, repair, and retrofit decisions. An important application is the rapid selection of efficient sites for removal, given the large number of watershed barriers (i.e., Is barrier-A or barrier-B the priority for restoration?). Another application is the computation of restoration benefits at a particular site to compare the relative effects of alternative management actions (e.g., Is a fish ladder or dam removal preferrable?). Dozens of studies globally have examined these questions with different methods and techniques (O'Hanley and Tomberlin 2005, Neeson et al. 2015, McKay et al. 2016). 

This document reviews the development of a framework for prioritizing barriers or quantifying benefits of restoration actions at a watershed-scale. The purpose of this model is to assess watershed scale habitat connectivity and quality for migratory organisms to inform restoration planning. The framework is adaptable to varying project timelines, scopes, and applications. The following issues are crucial to the **scoping and application** of this framework:  

* The focus is on migratory organisms moving upstream through a watershed (e.g., salmon seeking access to breeding habitat). Downstream movement is not considered to be limiting in this model framework and is neglected, although it can be an important limiting factor for some species.  
* The framework focuses on quantifying the accessibility of habitat to a focal taxa, and habitat quantity and quality can be incorporated through multiple methods.  
* Overall, this modeling approach may be used to rapidly screen barriers for site selection or quantify benefits of alternative restoration actions at a site scale. For simplicity, this document will refer to these methods as "barrier prioritization techniques".  
* The framework can be used in the context of benefits of barrier removal for desired taxa or impacts of barrier removal relative to invasive species. In either case, watershed connectivity may be assessed following this procedure, but the interpretation of results may be different. For instance, a user may chose to maximize connectivity for desirable species while minimizing connectivity for invasive species.  
* Many barrier prioritization studies address multiple taxa in development of watershed priorities. The framework could be applied to multiple taxa separately (i.e., developing connectivity estimates for each taxa), and then an overall assessment of could be obtained using a combined metric (e.g., sum or average of connected habitat for all species).

Given these scoping factors, the model will be referred to as the Watershed-Scale Upstream Connectivity Toolkit (WUCT).  This document intends to record the WUCT framework's technical details, use, and relevant information for the purpose of the USACE planning model certification requirement (EC 1105-2-412, PB 2013-02). Regional certification is requested, and a procedure is recommended for tailoring the framework for local applications.


## **Development Process**   

To develop the WUCT, a common ecological model development process of conceptualization, quantification, evaluation, and application was followed (Grant and Swannack 2008, Swannack et al. 2012). The modeling approach has been applied numerous times to USACE and non-USACE studies, and this document represents multiple iterations of this framework as well as refinements of the numerical code used. Prior applications and reviews include the following:  

* *USACE Truckee River Project* (Conyngham et al. 2011) - An initial model was developed in support of the Sacramento District's Truckee Meadows Flood Control and Ecosystem Restoration study (c. 2007-2008). The initial model considered a series of dams along the mainstem of the Truckee River and accounted for the cumulative effects of fish passage processes. This model was programmed in Microsoft Excel.  
* *Generic network algorithm* (McKay et al. 2013) - The Truckee River model was generalized by incorporating network analytic methods, which allowed for more complex watershed shapes. The generic model was applied in a theoretical context to identify key driving variables at large-scales as well as to the Truckee River application described above. This model was programmed in the R Statistical Software, and code was published as part of a peer-reviewed journal paper.  
* *Synthesis of barrier prioritization methods* (McKay et al. 2016) - A synthesis of barrier prioritization methods was conducted with an international, interagency working group to identify consistencies in methods for barrier prioritization. The basic framework applied here was conceptually proposed as part of this synthesis.  
* *USACE Hudson-Raritan Estuary case study* (McKay et al. 2017) - A preliminary application of this synthetic framework was conducted in the Harlem River, East River, and Western Long Island Sound Planning Region of the Hudson-Raritan Estuary study led by New York District. This application focused on an entire planning region with 49 barriers dispersed over multiple watersheds.  
* *USACE Proctor Creek Ecological Model* (McKay et al. 2018ab) - The framework of connectivity-weighted habitat quantity and quality was also applied in the Proctor Creek ecosystem restoration study in Atlanta, Georgia led by the Mobile District. This application focused more broadly on ecosystem-scale outcomes (rather than migratory fishes explicitly); however, the underlying connectivity algorithm was identical to the network analytic method (McKay et al. 2013). The Proctor Creek Ecological Model (PCEM) was certified by the ECO-PCX for use in this study.  


## **Conceptualization**   

Conceptual ecological models are useful for all USACE ecosystem restoration projects due to their capacity to increase understanding, identify potential alternatives, and facilitate team dialog (Fischenich 2008). Conceptual models also inform the development of quantitative ecological models (Grant and Swannack 2008, Swannack et al. 2012). The following text and figures present a few schematic conceptual models along with narratives describing the WUCT modeling framework. 

*Overarching Conceptual Structure*: An enormous array of models has been developed for stream corridor assessment and barrier prioritization. In addition to differing in technical complexity and application time, these models also vary based on factors such as the disciplinary perspective (e.g., hydrologic, geomorphic, ecological), the hierarchical level of ecological element addressed (e.g., individuals, populations, communities, ecosystems), the basic approach to modeling (e.g., statistical, theoretical), input requirements (e.g., few parameters vs. extensive geospatial layers), the treatment of time and space (e.g., lumped vs. distributed), and the degree of development (e.g., long history vs. new tool). Here, we take a common approach to ecological modeling based on quantity and quality of habitat. These "index" models (Swannack et al. 2012) were originally developed for species-specific applications (e.g., slider turtles), but the general approach has also been adapted to guilds (e.g., salmonids), communities (e.g., floodplain vegetation), and ecosystem processes (e.g., the Hydrogeomorphic Method).

This model framework emphasizes the restoration of hydrologic connectivity, which refers to the "water-mediated transfer of matter, energy, and/or organisms within or between elements of the hydrologic cycle" (Pringle 2001). In a recent review of barrier prioritization decision support models, McKay et al. (2016) identify three basic elements common to a diversity of connectivity prioritization applications: watershed connectivity / barrier passability, habitat quantity, and habitat quality. Each of these components can be assessed relative to multiple species (e.g., shad vs. eel), habitat requirements (e.g., tolerant vs. intolerant to pollution), life histories (e.g., anadromous vs. catadromous), and even units of measure (e.g., habitat quantity as length along a river vs. the wetted area of the river). Here, a simple view of each of these components was adopted that prioritizes reconnecting tributaries to the ocean (rather than reconnecting fragmented riverine stretches to one another), long reaches of rivers (over short), and patches of high quality habitat (over low).

*Habitat Quantity*: The WUCT focuses on the accesibility of habitat upstream of movement barriers, but the framework can accommodate any metric of habitat quantity into this assessment. The majority of barrier prioritization studies use river length as the focal habitat outcome (O'Hanley and Tomberlin 2005, Cote et al. 2009, McKay et al. 2013, King and O'Hanley 2014), although surface area (Brev? et al. 2014) or volume (Grill et al. 2014) have also been used to account for variation in width and depth of a river. For the purpose of the WUCT, all open river habitat upstream of a dam is associated with that structure. For instance, Figure 1 shows a small watershed with 11 nodes dividing the watershed, but the WUCT would only focus on three patches of habitat (i.e., the black patch downstream of all dams, the blue patch associated with Dam-A, and the purple patch associated with Dam-B).

*Habitat Quality*: The WUCT also presents a generic framework for assessing habitat quality. Habitat quality is consistently scaled from 0 to 1, and any family of methods may be used to quantify quality for a given patch. Figure 1 shows a conceptual example of how habitat quality can vary throughout a watershed. 

![Figure 1. Description of habitat for the WUCT: (left) habitat quantity associated with a given dam, and (right) habitat quality for a given patch.](WUCT_Fig1_Habitat.png)

*Connectivity and Passability*: A barrier prioritization could be made solely on the number of miles of river upstream of a structure.  However, a structure's passability and the cumulative passability of many structures may have dramatic effects on the outcome of a prioritization (O'Hanley and Tomberlin 2005). For instance, if barrier-A has 2 miles upstream and barrier-B has 10 miles upstream, a site-by-site scoring system would recommend the removal of barrier-B.  However, if barrier-A is downstream of barrier-B, watershed reconnection to the outlet first requires the removal of barrier-A. 

The "passability" of any given watershed barrier can first be considered in isolation of all other barriers. Passability refers to the proportion of taxa that are estimated to be able to move upstream of a structure, which may be assessed at the population or community scale (i.e., the proportion of chinook salmon or the percent of all species that can pass a barrier). Passability can be view on scales that are continuous (i.e., a range of values from zero to one), binary (i.e., passability is 0 or 1 for all structures), or categorical (i.e., all fish ladders have the same passability, which is different than all dam removals). Kemp and O'Hanley (2010) provide a thorough review of techniques for assessing passability, and McKay et al. (2016) review 46 barrier prioritization studies which apply a variety of these methods in the context of watershed-scale assessments.   

The sequential impacts of barriers has been well-described elsewhere (McKay et al. 2016), and a variety of metrics exist to quantify the effects of a barrier on watershed connectivity (e.g., O'Hanley and Tomberlin 2005, Cote et al. 2009, Martin and Apse 2011). The WUCT adopts a framework which combines the passability of all barriers in a watershed and focuses on the connectivity of the watershed outlet to any given reach of river.

Figure 2 presents example applications of how binary passage (top), partial passage (middle), and imperfect restoration of passability (bottom) could all be applied in the same watershed.

![Figure 2. Examples of how passability assumptions and methods translate in different restoration priorities: (top) assumes binary passage at all dams, (middle) assumes partial passage at dams, and (bottom) assumes that fish ladders denoted by hashed boxes lead to imperfect passage. Green, yellow, and red denote full, partial, and no connection to the outlet, respectively.](WUCT_Fig2_Connectivity.png)

*Flexible Framework*: The WUCT is designed as a flexible framework for combining the above factors as appropriate for a given application.  This "plug-and-play" or "mix-and-match" structure allows for teams to adjust methods based on their project objectives, time and resource constraints, data availability, and the ecological context of the watershed. Figure 2 provides three examples of how different approaches have been combined in prior applications.

![Figure 3. Examples of project-specific combination of the three primary components of the WUCT framework (passability, habitat quantity, and habitat quality). Applications to date are used as example of how the framework may be adapted to different project objectives, data availability, and timelines: Truckee River (Conyngham et al. 2011 shown in yellow), Proctor Creek (McKay et al. 2018ab shown in blue), and preliminary Hudson-Raritan planning region analysis (McKay et al. 2017 shown in orange).](WUCT_Fig3_Methods.png)


## **Quantification**   

Barrier prioritization has become a focus in river restoration and watershed management with more than 50 studies globally (see review by McKay et al. 2016). Many of these studies apply similar frameworks for assessing watershed connectivity, and the WUCT directly builds from these existing approaches. This framework relies on three key ideas: (1) the shape / topology of the watershed is crucial to tracking the dependency between actions, (2) the passability of one structure and cumulative passability of many structures affect priorities, and (3) the quantity and quality of habitat accessible is an important weighting factor. 

The outcome of the WUCT is a metric of total accessible habitat, which incorporates connectivity, quantity, and quality. The maximum value for the metric is the total quantity of habitat for a watershed, and the minimum value of the metric is zero. For instance, a watershed with 15 miles of habitat could have a total accesible habitat ranging from 0-15. The range of values is determined by both connectivity and habitat quality. If 5 miles of the watershed had extremely poor quality (i.e., 0) and perfect connectivity throughout the watershed (i.e., 1), then the metric would be 10 miles of total accessible habitat.  If 5 miles of the watershed had extremely poor quality (i.e., 0) and the entire watershed was partially disconnected at the outlet of the watershed (i.e., passability = 0.5), then the metric would be 5 miles of total accessible habitat. In other words, this metric provides a connectivity- and quality-weighted metric of total habitat.


#### **Model Theory and Code**  

The graph theoretical basis of the WUCT is reviewed in detail elsewhere (McKay et al. 2013), but the methods can be summarized briefly as follows. An adjacency matrix is used to summarize the shape and topology of the watershed, and passability at each node is provided as a vector. The adjacency matrix and passability vector may combined through matrix multiplication to assess the "passage matrix", which is a summary of the cumulative connectivity from the outlet to every other node in the watershed. The passage matrix may be multipled by a vector of habitat (including quantity and quality) to arrive at the total accessible habitat in the watershed. The code below creates a function to conduct these steps numerically in the R Statistical Software.

![Figure 4. Calculation method applied in the WUCT (from McKay et al. 2013). Simplified network to demonstrate computation of the habitat connectivity index for upstream passage. Total accessible habitat is the output of this model, rather than the normalized metric shown in this figure.](WUCT_Fig4_EcoApps.png)


An adjacency matrix is a square n X n matrix summarizing the topology of the watershed by specify which nodes are connected to other nodes. A "1" represents connectivity between two nodes, and a "0" represents no connectivity. The columns of the matrix represent downstream locations and the rows represent upstream location. Thus, any row-column pairing defines whether two nodes are connected. For instance, Figure 4 shows that row-column location (2,1) is connected, which implies that downstream node-1 (specified by column 1) is connected to upstream node-2 (specified by row 2). Importantly, adjacency matrices can be used to specify upstream and downstream connectivity, but for the WUCT, only upstream connectivity should be specified (i.e., there should be only zeros above the diagonal of the matrix). 


```{r}
#Initialize the modeling environment by clearing local memory
rm(list=ls(all=TRUE)) 

##########
#WRITE NEW FUNCTION - WATERSHED CONNECTIVITY

connectivity <- function(n,A,pass,hab){
  #Combine network morphology, passage rates, and dam configuration into a "passage matrix"
  #
  #Args: 
  #   Variables are defined as follows:
  #   n: numeric; number of nodes in the watershed
  #   A: matrix; adjacency matrix summarizing watershed topology
  #   pass: numeric; vector of passage rates at each node
  #   hab: numeric; vector of habitat directly above each node (including quantity and quality)
  #
  #Returns:
  #   numeric; total accessible habitat score
  
  #Create an empty upstream passage matrix
  Ptemp <- matrix(0,nrow=n,ncol=n)
  
  #Populate passage rates throughout the network (passage matrix)
  for(i in 1:n){
    Ptemp[i,] <- A[i,] * t(pass)
  }	
  
  #Compute the cumulative passage rate to the node
  Con <- solve(diag(n) - Ptemp)
  
  #Compute the cumulative passage rate beyond the node
  cpass <- Con[,1] * pass
  
  #Incorporate habitat quantity and quality into the metric
  habtot <- sum(cpass * hab)
  
  #Return the final output list (total accesible habitat)
  habtot
}

```

#### **Model Parameterization**  

There are four input variables to this function:  

* n = number of nodes in the watershed  
* A = adjacency matrix summarizing watershed topology (n X n in size)  
* pass = vector of passage rates at each node (n in length)  
* hab = vector of habitat directly above each node including quantity and quality (n in length)

These inputs do not explicitly incorporate time or habitat quality. However, the function could be applied to incorporate these concepts. For instance, if habitat quality varied through time, the function could be applied iteratively with a changing "hab"" vector to assess changes through time. The function is explicitly designed to be scalable to any size watershed, but matrix functions in watersheds with more than 100 nodes may be computationally intensive.

Manual entry of the model inputs may be sufficient for small watersheds. However, data management becomes challenges as watershed size and the number of alternatives increases. The model application section describes a standardized data structure developed for the Hudson-Raritan Estuary project, which can flexibly manage larger numerical problems.

The flexibility of this framework allows users to parameterize the model in many unique ways depending upon their application. The flexibility and generality of the WUCT is an asset from a numerical standpoint. However, there is the possibility for inappropriate parameterization of any one component. For instance, Figure 3 shows three applications of this framework, which contain very different parameterizations. Given this flexibility, model application should be reviewed carefully during Agency Technical Review (ATR) or Independent External Peer Review (IEPR) to ensure appropriate use. For simple or rapid applications, a lower level of resolution or data may be appropriate, whereas large investments or long-term projects may require more refined inputs. In particular, three key issues should be scrutinized relative to all applications:  

* Passability: Were structure passabilities considered at the population, guild, or community level? Were binary, continuous, or categorical approaches used? Were passage rates based on empirical data, analytical methods, or professional judgment?   
* Habitat quantity: What metric of habitat quantity was applied (length, area, volume)? Is this metric appropriate for the project objectives?   
* Habitat quality: Is habitat quality included in the analysis (quantity only analyses may be appropriate)? How was quality assessed? What methods or data were used to describe quality? Were they appropriate for the spatial scale of this model application?  


Network visualization is often useful as a mechanism to ensure appropriate inputs and summarize outputs. A variety of R-packages are available to facilitate visualization, and users are encouraged to examine igraph, ggnet2, and networkD3 or explore other options in the r-graph-gallery (https://www.r-graph-gallery.com/).

## **Evaluation**   

Model evaluation is the process to ensure that numerical tools are scientifically defensible and transparently developed. Evaluation is often referred to as verification or validation, but in fact includes a family of methods ranging from peer review to model testing (Schmolke et al. 2010). The USACE has established an ecological model certification process to ensure that planning models used on ecosystem restoration projects are sound and functional, which generally consists of evaluating tools relative to three categories: technical quality, system quality, and usability (EC 1105-2-412, PB 2013-02).

#### **Technical Quality**   

The technical quality of a model is assessed relative to its reliance on contemporary theory, consistency with design objectives, and degree of documentation and testing. As described in the conceptualization and quantification sections, the WUCT framework has been extensively applied for barrier prioritization through a long, peer-reviewed development history, including conceptual development (McKay et al. 2016), theoretical applications (McKay et al. 2013), and restoration applications (Conyngham et al. 2011, McKay et al. 2017, McKay et al. 2018ab). 

#### **System Quality**   

Ecological models must not only maintain an appropriate theoretical and technical basis but also be computationally accurate.  System quality refers to the computational integrity of a model (or modeling system).  For instance, is the tool appropriately programmed, has it been verified or stress-tested, and do outcomes behave in expected ways?  The system quality of these models was evaluated in a variety of ways, including both code development best practices and model testing. Code was adopted and modified from a previous study (McKay et al. 2013) to minimize new computational errors. All code was error-checked during and after development by the primary programmer (McKay) and inspected by various teams during prior uses.  Error checking considered consistent variable naming, investigated outputs from each line of code, and blocks of code (e.g., loops).

Numerous model outputs were examined thoroughly as test cases for model functionality. The first test case was drawn directly from McKay et al. (2013). Figure 4 presents an example basin along with manual computation of accessible habitat. The computation function produced the expected 4.5 miles of accesible habitat.

```{r}
#FUNCTION TESTING - Example from McKay et al. (2013)
#Parameterized as shown in Figure 4 
Atest <- rbind(c(0,0,0,0,0,0,0,0,0),c(1,0,0,0,0,0,0,0,0), c(0,1,0,0,0,0,0,0,0),
           c(0,1,0,0,0,0,0,0,0), c(0,0,0,1,0,0,0,0,0), c(0,0,0,0,1,0,0,0,0),
           c(0,0,0,0,1,0,0,0,0), c(0,0,0,0,0,0,1,0,0), c(0,0,0,0,0,0,0,1,0))
passtest <- c(1, 1,1,0.5,0.5,1,1,1,1)
habtest <- c(1,2,0,1,2,0,1,1,0)
ntest <- length(passtest)
connectivity(ntest,Atest,passtest,habtest)

```

The sample basin from Figure 4 was also used to test that changes in passage rates resulted in expected changes in accessible habitat. If the passage rate at node-4 equals 0, then there should be only 3 miles of accessible habitat.  The model produced the expected outcome.

```{r}
#FUNCTION TESTING - Example from McKay et al. (2013)
#Parameterized as shown in Figure 4 with node-4 passability = 0
passtest <- c(1, 1,1,0,0.5,1,1,1,1)
connectivity(ntest,Atest,passtest,habtest)

```

The sample basin from Figure 4 was also used to test that changes in cumulative passage rates resulted in expected changes in accessible habitat. If the passage rate at node-4 equals 0.25, then there should be 3.75 miles of accessible habitat.  The model produced the expected outcome.

```{r}
#FUNCTION TESTING - Example from McKay et al. (2013)
#Parameterized as shown in Figure 4 with node-4 passability = 0
passtest <- c(1, 1,1,0.25,0.5,1,1,1,1)
connectivity(ntest,Atest,passtest,habtest)

```

The sample basin from Figure 1 was also used to test that changes in the adjacency matrix produced appropriate outcomes. Habitat was assumed to be 7, 3, and 2 units above the terminus, Dam-A, and Dam-B, respectively. If both dams have a passage rate of 0.5, then there should be 9.0 miles of accessible habitat.  The model produced the expected outcome.

```{r}
#FUNCTION TESTING - Example from Figures 1 and 2
#Parameterized as passage rate = 0.75 for all node
Atest2 <- rbind(c(0,0,0),c(1,0,0),c(0,1,0))
passtest2 <- c(1,0.5,0.5)
habtest2 <- c(7,3,2)
ntest2 <- length(passtest2)
connectivity(ntest2,Atest2,passtest2,habtest2)
```

The sample basin from Figure 1 was also used to test changes in passage rates. If Dam-A were removed (i.e., passage = 1), then the basin would have 11 miles of accessible habitat.  The model produced the expected outcome.

```{r}
#FUNCTION TESTING - Example from Figures 1 and 2
#Parameterized as passage rate = 1 for Dam-A and passage rate = 0.5 for Dam-B
Atest2 <- rbind(c(0,0,0),c(1,0,0),c(0,1,0))
passtest2 <- c(1,1,0.5)
habtest2 <- c(7,3,2)
ntest2 <- length(passtest2)
connectivity(ntest2,Atest2,passtest2,habtest2)
```

#### **Usability**   

The usability of a model can influence the repeatable and transparent application of a tool. This type of evaluation typically examines the ease of use, availability of inputs, transparency, error potential, and education of the user. As such, defining the intended user(s) is a crucial component of assessing usability. The WUCT was developed for flexible application by USACE technical teams across a range of ecosystems, contexts, and uses. Currently, there is no graphical user interface (GUI) for the model beyond the scripted function itself. 

The function is written in the R statistical software, which is an ACE-IT approved, free platform for conducting a variety of statistical and numerical analyses. The software can be accessed through two user interfaces, "base R" or "RStudio". While only maintained in script format, the current form of the model has maintained usability through a few key mechanisms. First, all code is well-documented for easy understanding by future users. Second, the model is designed in a simple input-output workflow where any vector or matrix can be used to parameterize the model. Third, an example input file is provided below as a template for users. Fourth, model inputs may be obtained from a variety of sources to maintain data availability and easy processing. Currently, training is unavailable for the WUCT. However, the development team is willing to support future applications and provide technical support to other users, as needed.

The current script-based format of the model requires familiarity with R. As such, the model is recommended for use by intermediate-level R users. Future versions will pursue a GUI to facilitate use by non-R users.


#### **Other Considerations**   

Other factors may be important to model evaluation in addition to the factors described above. The following issues often arise during model certification:  

* USACE policy compliance. The WUCT is compliant with USACE policies for applications in aquatic ecosystem restoration. Social and economic factors are not included in the model, and the outputs are non-monetary in nature.  
* End-user capability. The WUCT represents a common set of methods applied by a diversity of groups interested in watershed connectivity and barrier prioritization. The script-based format of the model requires user-familiarity with the R programming language and may not be easily accessible by all USACE staff. However, USACE personnel are increasingly becoming aware of the R language, and its application is growing as universities are teaching more analytical and programming skills in both undergraduate and graduate programs. The development team is also willing to support applications by other users and/or provide background on R programming.  
* Adaptability for future applications. The WUCT is extremely adaptable to a variety of data inputs, applications, and modifications. The compact, script-based format also makes the WUCT easily adaptable to include future improvements (e.g., bidirectional movement).  
* Model archival. The integrated documentation and code presented here is easily archivable, and the web-ready html formatting of this document facilitate web-archival directly on the USACE restoration model library.


## **Application**

This section presents two examples of how the WUCT may be applied to assess restoration priorities and compare restoration alternatives. Other example applications may be found for the Truckee River (McKay et al. 2013), the Hudson-Raritan Estuary (McKay et al. 2017), and Proctor Creek (McKay et al. 2018ab). A step-by-step user's guide for WUCT application is presented in Appendix A.


#### **Application 1: Dummy Network from McKay et al. (2013)**


This example project builds from the sample basin presented in Figure 4. This hypothetical watershed has two dams (Node-4 and Node-5), which are assumed to be impermeable barriers (i.e., passability=0). A hypothetical set of restoration actions may include:  

* Installation of a fish ladder at Node-4, which is assumed to increase passability to 50%.  
* Removal of the dam at Node-4, which is assumed to increase passability to 100%.  
* Improvement of habitat above Node-4 from a quality of 0.5 to 1.0.  
* Installation of a fish ladder at Node-5, which is assumed to increase passability to 50%.  
* Removal of the dam at Node-5, which is assumed to increase passability to 100%.

Each combination of these five restoration actions was examined as a potential restoration plan. While manual entry of model inputs may be sufficient for small watersheds, data management becomes challenges as watershed size and the number of alternatives increases. As an example of this increasing problem size, a standardized data structure was developed, which can flexibly manage larger numerical problems. Table 1 summarizes the site-scale restoration alternatives considered and serves as a generic input format (for adaptation by future applications).

```{r}
#Dummy example - Import data
barrieralts <- read.csv("WUCT_TestData_2018-06-28_BarrierAlts.csv", header=TRUE, dec=".")
A <- data.matrix(read.csv("WUCT_TestData_2018-06-28_Adjacency.csv", header=FALSE, dec="."))

##########
#Send input data from the example problem as a table
knitr::kable(barrieralts, caption="Table 1. Sample input matrix for rapid application of the WUCT to the example basin from Figure 4. Each node must specify the existing condition (shown as Alt=0), and multiple restoration alternatives can also be specified for each node (shown as Alt>0). For instance, Node-4 shows competing changes in the passability of this node with the existing condition totally impassable, a partial improvement in passability (e.g., from a fish ladder), a full improvement of passability (e.g., from dam removal), improvements in habitat quality, and all combination of these actions at Node-4.")

```

The following script isolates the necessary input data, computes all permutations of restoration actions, and computes connectivity for each watershed-scale restoration plan. Table 2 presents the output of the WUCT in this application. The connectivity values represent a connectivity- and quality-weighted assessment of total habitat at the watershed scale. These plans provide a range of ecological outcomes from the existing, disconnected condition (Plan 1) to complete reconnection of the watershed and enhancement of site quality (Plan 18). Each plan could be coupled with cost estimates to inform cost-effectiveness and incremental cost analyses (CE-ICA, Robinson et al. 1995).

```{r}
#Isolate properties of the alternatives at each barrier
bnames <- paste(unique(barrieralts$BarrierID))
nbar <- length(bnames)

##########
#Compute the number of alternatives at each barrier
nalts <- c()
for(i in 1:nbar){nalts[i] <- length(which(barrieralts$BarrierID == bnames[i]))}
nalts.total <- prod(nalts) #Compute the total number of combinations of actions

##########
#Create a list which stores all alternatives at each site
  #This list stores the row number of each alternative from "barrieralts".
site.alts <- list() #Create an empty list to store the combinations of sites/alts
for(i in 1:nbar){site.alts[i] <- list(which(barrieralts$BarrierID == bnames[i]))}

#Compute all combinations of alternatives and sites
  #This returns a data frame where each columns is a site or barrier and each row is a unique plan with a combination of sites and alternatives.
sitealts.combos <- expand.grid(site.alts)

#Convert this data frame to a matrix 
sitealts.combos <- data.matrix(sitealts.combos)

##########
#Compute connectivity for each plan
WC.out <- c() #Empty vector to store watershed connectivity for each plan
for(i in 1:nalts.total){
  #Isolate the passability and habitat values to be used
  pass.temp <- barrieralts$Passability[sitealts.combos[i,]]
  hab.temp <- barrieralts$QualityUpstream[sitealts.combos[i,]] * barrieralts$QuantityUpstream[sitealts.combos[i,]]
  
  #Compute connectivity and store result
  WC.out[i] <- connectivity(nbar,A,pass.temp,hab.temp)
}

##########
#Send output from the example problem as a table
WC.example <- matrix(NA, nrow=nalts.total,ncol=nbar+2)
for(i in 1:nalts.total){WC.example[i,] <- c(i,barrieralts$Alt[sitealts.combos[i,]],WC.out[i])}
colnames(WC.example) <- c("Plan",bnames,"Connectivity")
knitr::kable(WC.example, caption="Table 2. Alternatives analysis using the WUCT and the example basin from Figure 4. Competing restoration alternatives are shown as rows, and the alternative used in the plan is denoted by the number shown.")

```


#### **Application 2: Bronx River**

This model application examines fish passage prioritization for the Hudson-Raritan Estuary (HRE) project led by the USACE New York District (NAN). Specifically, this application quantifies fish passage benefits associated with two sites in the Bronx River watershed, which were proposed in the 2017 Draft Feasibility Report (USACE 2017) and supported by the Comprehensive Restoration Plan (USACE 2016). Fish passage outputs are quantified in terms of "accessible habitat" using the Watershed-Scale Upstream Connectivity Toolkit (WUCT) with river herring as the focal taxa. 

Three dams are of interest in the Bronx River system moving from downstream to upstream. First, the East 182nd Street Dam is the first barrier encountered, where a fish ladder (Alaskan Steep pass) has been constructed by partners including NYC Parks, the Bronx Borough, the Wildlife Conservation Society, the National Oceanic and Atmospheric Administration, the US Fish and Wildlife Service, New York State, and the National Fish and Wildlife Foundation (Lumbian and Larson 2015). Second, the Bronx Zoo Dam is the next structure, where USACE has proposed three restoration alternatives as part of this feasibility study (all including a fish ladder). Third, the Stone Mill Dam (aka. Snuff Mill Dam) is the next structure, where USACE has proposed three restoration alternatives as part of this feasibility study (one with a fish ladder + attractors, one with a fish ladder, and one without a fish ladder). A significant amount of habitat is accessible above the Stone Mill Dam, when considering both main steam and tributary habitats. The Bronx River has been shown to support river herring populations, where accessibility is not limiting (Larson and Sugar 2004), and the 182nd St Dam fish ladder has subsequently demonstrated that river herring will utilize technical fishways in this region.

The WUCT requires three general types of inputs, and the HRE parameterization is as follows (also shown in Table 3):  

* *Habitat Quantity* - For each barrier, an area of upstream habitat opened is used as the primary basis for habitat quantity. First, the length of upstream habitat was computed (i.e., the distance between the dam and the next upstream barrier) and included all tributary habitats that would be newly accessible. River width was estimated from aerial photos in Google Earth as the smallest observable width upstream of the structure (i.e., an extremely conservative estimate). To create an area-based metric , the length of river is multiplied by width.  
* *Habitat Quality* - Habitat quality was predicted based on a watershed-scale, geospatial analysis of all upstream habitat as presented in McKay et al. (2017). The model included three metrics accounting for land use development pressure, water quality, and the proportion of the basin in conservation status.   
* *Passability* - Local studies of fish passage rates (i.e., passability) are unavailable in the Bronx River. As such, passability was estimated based on studies elsewhere on the efficacy of technical fishways in general and for river herring specifically. Prior to restoration actions, all structures are assumed to have zero passage of river herring. Alewife studies in New England (Franklin et al. 2012) report high overall passage rates of 64-99% passage in the East River, Massachusetts with particularly high rates for technical steep passes of 94-97%.  These values are in line with a meta-analysis of 65 published fish passage studies (Noonan et al. 2011), which indicated that fish passage efforts typically result in 42% fish passage on average across a variety of taxa. Based on the Massachusetts data, we used 80% passability for all fish ladders as a conservative estimate of passage efficiency. The 182nd Street Dam and Ladder were included in all analyses as part of the future without prokect condition. At Stone Mill Dam, Alternative-A includes fish attractors to increase utilization of the ladder, which were assumed to increase passability by 10% (i.e., to 88% total).

The following script imports data, isolates the necessary input data, computes all permutations of restoration actions, and computes connectivity for each watershed-scale restoration plan. Table 4 presents the output of the WUCT in this application. The connectivity values represent a connectivity- and quality-weighted assessment of total habitat at the watershed scale. 

```{r}
#Dummy example - Import data
barrieralts <- read.csv("WUCT_HREData_2018-07-18_BarrierAlts.csv", header=TRUE, dec=".")
A <- data.matrix(read.csv("WUCT_HREData_2018-07-18_Adjacency.csv", header=FALSE, dec="."))

##########
#Send input data from the example problem as a table
knitr::kable(barrieralts, caption="Table 3. Input data for the WUCT to the Bronx River as described in the text. Each node specifies the existing condition (shown as Alt=0), and multiple restoration alternatives are also be specified for each node (shown as Alt>0).")
```


```{r}
#Isolate properties of the alternatives at each barrier
bnames <- paste(unique(barrieralts$BarrierID))
nbar <- length(bnames)

##########
#Compute the number of alternatives at each barrier
nalts <- c()
for(i in 1:nbar){nalts[i] <- length(which(barrieralts$BarrierID == bnames[i]))}
nalts.total <- prod(nalts) #Compute the total number of combinations of actions

##########
#Create a list which stores all alternatives at each site
  #This list stores the row number of each alternative from "barrieralts".
site.alts <- list() #Create an empty list to store the combinations of sites/alts
for(i in 1:nbar){site.alts[i] <- list(which(barrieralts$BarrierID == bnames[i]))}

#Compute all combinations of alternatives and sites
  #This returns a data frame where each columns is a site or barrier and each row is a unique plan with a combination of sites and alternatives.
sitealts.combos <- expand.grid(site.alts)

#Convert this data frame to a matrix 
sitealts.combos <- data.matrix(sitealts.combos)

##########
#Compute connectivity for each plan
WC.out <- c() #Empty vector to store watershed connectivity for each plan
for(i in 1:nalts.total){
  #Isolate the passability and habitat values to be used
  pass.temp <- barrieralts$Passability[sitealts.combos[i,]]
  hab.temp <- barrieralts$QualityUpstream[sitealts.combos[i,]] * barrieralts$QuantityUpstream[sitealts.combos[i,]]
  
  #Compute connectivity and store result
  WC.out[i] <- connectivity(nbar,A,pass.temp,hab.temp)
}

##########
#Send output from the example problem as a table
WC.HRE <- matrix(NA, nrow=nalts.total,ncol=nbar+2)
for(i in 1:nalts.total){WC.HRE[i,] <- c(i,barrieralts$Alt[sitealts.combos[i,]],WC.out[i])}
colnames(WC.HRE) <- c("Plan",bnames,"Connectivity")
knitr::kable(WC.HRE, caption="Table 4. Alternatives analysis using the WUCT in the Bronx River Watershed. Competing restoration alternatives are shown as rows, and the alternative used in the plan is denoted by the number shown.")

```


## **Communication**

The four phases of ecological model development presented here (i.e., conceptualization, quantification, evaluation, and application) are crucial to understanding the theoretical, practical, and applied uses of models for restoration projects. However, model development and application also requires communication of each of these model development phases. The WUCT has sought to transparently communicate the theory and application of this toolkit in the following ways:  

* Published papers. As described, multiple peer-reviewed journal papers and government reports have been released, which detail the various theoretical, conceptual, and numerical aspects of the WUCT. These reports serve to communicate the approach both internal and external to USACE as well as providing a mechanism for verifying the scientific validity of the approach.   
* Annotated code. Future application of a model is facilitated by users being able to follow the logic of the tool. This is particularly important for models without GUIs such as the WUCT. All applications of the model have extensively documented script at a line-by-line level.   
* Output. Model outputs are formatted for easy import into subsequent analyses, such as CE-ICA. 


## **Summary**

This model document describes a procedure for quantifying benefits associated with removal of organism movement barriers within a watershed (e.g., dam removal, culvert repair, fish ladder installation) or impacts of barrier addition (e.g., dam construction, weir installation). The model focuses on upstream movement of migratory organisms such as fish and operates at the watershed-scale. The algorithm is based on four primary components: habitat quantity upstream of a dam, habitat quality upstream of a dam, the passability of a structure for a given organism, and the shape/topology of the watershed. This algorithm combines these data to estimate quality-weighted, accessible habitat at the watershed scale. 

This modeling framework has been applied to multiple USACE projects with varying input parameters and ecological contexts. A few notable **limitations and assumptions** include the following issues. Future improvements to the WUCT should consider addressing these limitations.  

* Upstream only movement. The current model only incorporates movement upstream, which implicitly assumes downstream movement is not a limiting factor for the organism's life cycle. However, downstream movement can be an important driver of ecological function (e.g., fish trapping in agricultural diversion ditches or fish mortality due to hydropower turbines). Other connectivity algorithms can incorporate bidirectional movement, but are not currently incorporated into the WUCT.  
* The WUCT is primarily applicable in the context of migratory fishes because of the focus on connectivity to the outlet of the system. Alternative connectivity algorithms exist for resident fishes (e.g., Cote et al. 2009).  
* Numerical limits. The basic WUCT algorithm and the example application are extremely flexible for large-scale examination of watershed priorities and benefits. However, numerical problem size increases rapidly with the number of barriers are alternatives. For instance, a watershed with 30 barriers considering only the presence or absence of the barriers could result in more than a billion combinations of actions. As such, care should be taken to limit problem size to the extent practicable.  
* User interface. The lack of a user interface limits the applicability of the toolkit to those familiar with the R programming language.

As presented, the WUCT is a simple, flexible framework for quantifying the connectivity of watersheds and potential improvements through restoration actions. The adaptability of the framework is a primary strength of the tool, but this flexibility could also leads to significant variation in its application. Recommendations for review of each application have been provided to facilitate consistent and appropriate application in USACE planning.  


## **References**

Bellmore J.R., Duda J.J., Craig L.S., Greene S.L., Torgersen C.E., Collins M.J., and Vittum K. Status and trends of dam removal research in the United States. WIREs WATER, doi: 10.1002/wat2.1164.

Brev? NWP, Buijse AD, Kroes MJ, Wanningen H, Vriese FT. 2014. Supporting decision-making for improving longitudinal connectivity for diadromous and potamodromous fishes in complex catchments. Science of the Total Environment 496: 206-218.

Conyngham J., McKay S.K., Fischenich C., and Artho D. 2011. Environmental benefits analysis of fish passage on the Truckee River, Nevada: A case study of multi-action-dependent benefits quantification. ERDC TN-EMRRP-EBA-06. U.S. Army Engineer Research and Development Center, Vicksburg, Mississippi. 

Cote D., Kehler D.G., Bourne C., and Wiersma Y.F.  2009.  A new measure of longitudinal connectivity for stream networks.  Landscape Ecology, 24, 104-113.

Fischenich, J. C. 2008. The application of conceptual models to ecosystem restoration. ERDC TN-EBA-TN-08-1. Vicksburg, MS: U.S. Army Engineer Research and Development Center.

Franklin A.E., Haro A., Castro-Santos T., and Noreika J. 2012. Evaluation of nature-like and technical fishways for the passage of alewives at two coastal streams in New England. Transactions of the American Fisheries Society, 141, 624-637.

Grant W.E. and Swannack T.M.  2008.  Ecological modeling: A common-sense approach to theory and practice. Malden, MA: Blackwell Publishing.

Grill G, Dallaire CO, Chouinard EF, Sindorf N, Lehner B. 2014. Development of new indicators to evaluate river fragmentation and flow regulation at large scales: a case study for the Mekong River Basin. Ecological Indicators 45: 148-159.

Januchowski-Hartley S.R., McIntyre P.B., Diebel M., Doran P.J., Infante D.M., Joseph C., and Allan J.D.  2013.  Restoring aquatic ecosystem connectivity requires expanding inventories of both dams and road crossings.  Frontiers in Ecology and the Environment, 11 (4), 211-217.

Kemp P. and O'Hanley J. 2010. Procedures for prioritizing the removal of fish migration barriers: A review and synthesis. Fisheries Management and Ecology [Online] 17:297-322. Available at: http://dx.doi.org/10.1111/j.1365-2400.2010.00751.x.

King S. and O'Hanley J.R. 2014. Optimal fish passage barrier removal: revisited. River Research and Applications. DOI: 10.1002/rra.2859.

Larson M. and Sugar D. 2004. Phase 1 final report: Fish passage needs and feasibility assessment.  Natural Resources Group, Parks and Recreation, City of New York.  

Lumbian S. and Larson M. 2015. Final report on fish passage construction at the East 182nd Street Dam, Bronx River. New York City Parks.

Martin EH and Apse CD 2011. Northeast aquatic connectivity: an assessment of dams on northeastern rivers. The Nature Conservancy, Eastern Freshwater Program.

McKay S.K., Schramski J.R., Conyngham J.N., and Fischenich J.C.  2013.  Assessing upstream fish passage connectivity with network analysis.  Ecological Applications, 23 (6), 1396-1409.  doi: 10.1890/12-1564.1.

McKay S.K., Cooper A., Diebel M., Elkins D., Oldford G., Roghair C., and Wieferich D.  2016.  Informing watershed connectivity barrier prioritization decisions: A synthesis.  River Research and Applications, doi: 10.1002/rra.3021.

McKay S.K., Reif M., Conyngham J.N., and Kohtio D. 2017. Barrier prioritization in the tributaries of the Hudson-Raritan Estuary. ERDC TN-EMRRP-SR-82. U.S. Army Engineer Research and Development Center, Vicksburg, Mississippi. 

McKay S.K., Pruitt B.A., Zettle B., Hallberg N., Hughes C., Annaert A., Ladart M., and McDonald J.  2018a.  Proctor Creek Ecological Model (PCEM): Phase 1 Site screening.  ERDC TR-EL.  U.S. Army Engineer Research and Development Center, Vicksburg, Mississippi.  

McKay S.K., Pruitt B.A., Zettle B.A., Hallberg N., Moody V., Annaert A., Ladart M., Hayden M., and McDonald J.  2018b.  Proctor Creek Ecological Model (PCEM): Phase 2 benefits analysis.  ERDC TR-EL.  U.S. Army Engineer Research and Development Center, Vicksburg, Mississippi.  

Neeson T.M., Ferris M.C., Diebel M.W., Doran P.J., O'Hanley J.R., and McIntyre P.B.  2015.  Enhancing ecosystem restoration efficiency through spatial and temporal coordination.  Proceedings of the National Academies of Science, 112 (19), 6236-6241.

Noonan M.J., Grant J.W.A., and Jackson C.D. 2011. A quantitative assessment of fish passage efficiency. Fish and Fisheries, doi: 10.1111/j.1467-2979.2011.00445.x.

O'Hanley J.R. and Tomberlin D.  2005.  Optimizing the removal of small fish passage barriers.  Environmental Modeling and Assessment, 10, 85-98.

Pringle C.M.  2001.  Hydrologic connectivity and the management of biological reserves: A global perspective.  Ecological Applications, 11 (4), 981-998.

R Development Core Team.  2014.  R: A language and environment for statistical computing.  R Foundation for Statistical Computing, Vienna, Austria.  www.R-project.org.  

Robinson R. Hansen W., and Orth K.  1995.  Evaluation of environmental investments procedures manual interim: Cost effectiveness and incremental cost analyses.  IWR Report 95-R-1.  Institute for Water Resources, U.S. Army Corps of Engineers, Alexandria, Virginia.  

Swannack T.M., Fischenich J.C., and Tazik D.J.  2012. Ecological Modeling Guide for Ecosystem Restoration and Management.  ERDC/EL TR-12-18.  U.S. Army Engineer Research and Development Center, Vicksburg, MS.

U.S. Army Corps of Engineers (USACE). 2000. Planning Guidance Notebook. ER-1105-2-100. U.S. Army Corps of Engineers, Washington, D.C.

U.S. Army Corps of Engineers (USACE).  2011.  Assuring quality of planning models.  EC-1105-2-412. Washington, DC.  

U.S. Army Corps of Engineers (USACE). 2016. Hudson-Raritan Estuary: Comprehensive restoration plan. Volume 1, Version 1.0. New York District, U.S. Army Corps of Engineers, New York, New York.

U.S. Army Corps of Engineers (USACE). 2017. Hudson-Raritan Estuary Ecosystem Restoration Feasibility Study.  February 2017. New York District, U.S. Army Corps of Engineers, New York, New York.


## **Appendix A: User's Guide**

The following steps and guiding questions serve as a preliminary user's guide to application of the WUCT.  

* Establish restoration and modeling objectives. What are the needs for watershed connectivity modeling in terms of focal taxa, habitat requirements, data resolution needs, and time/resource constraints?   
* Conceptualize the ecological details of the model application. What are appropriate metrics for habitat quantity and quality? What is the availability of data or models to inform passability?  
* Quantify inputs.  How will inputs be quantified? Are additional models needed to quantify inputs (e.g., a habitat suitability model)?  How will the adjacency matrix be developed?     
* Import data to R. How will data be brought into the modeling environment (e.g., manual input vs. imported csv)?   
* Process data into correct format. As described the function input are in the form of vectors and matrices with an equal length of nodes.    
* Execute function. The "connectivity" function developed here may be applied in a single line of code or in a looped format for an iterative consideration of many plans such as the demonstrated application.  
* Store results.  How will outputs be stored for future use (e.g., in CE-ICA)? Will data be exported to another program? Will summary figures be developed within R?


## **Appendix B: Acknowledgements**

This model was developed over time with funding through multiple projects. An initial model supported the USACE Sacramento District Truckee Meadows Flood Control Project, which was developed with input from Jock Conyngham, Craig Fischenich, Leigh Skaggs, and Dan Artho. The model was generalized with support from the Ecosystem Management and Restoration Research Program (EMRRP), and John Schramski was crucial to this expansion. Applications of this model framework were also used in Proctor Creek, Georgia (with Tom Prebyl, Brian Zettle, and others at Mobile District), Duck River, Tennesee (with Tom Prebyl), and the Hudson-Raritan Estuary (with Jock Conyngham, Molly Reif, and Diana Kohtio). Dozens of USACE and external reviewers and managers have improved this model and associated applications through input and review. This model certification document was supported by USACE New York District's Hudson-Raritan Estuary and Hudson River Habitat restoration programs. The project manager for these projects is Ms. Lisa Baron, and the environmental lead for these efforts is Mr. Peter Weppler. Input and funding from these diverse sources over the course of 10+ years is greatly appreciated and acknowledged.


## **Appendix C: Model Certification Review**


Review comments were received on September 7, 2018. All comments follow the four-part comment structure of: identify the problem, describe the technical basis for the comment, rate the significance or impact of the problem, and recommend a mechanism for resolution. For brevity, on the text of the comments is included here.


#### **Reviewer 1: Michael Dougherty (USACE Rock Island District)**


The author appreciates the reviewer's time, input, and sample code. The review comments center largely on making the WUCT more generic for publication as an R-package via the CRAN network. However, as the reviewer acknowledges, review by CRAN can be time-consuming, and model development has been conducted under limited project planning timelines. CRAN review and packing will be pursued for future versions of the model, and the reviewer's sample code will provide an important template for future model development.

*Comment 1: Create an R package and submit to CRAN.*  

* Basis: The standard method with the R community for making R code available to a broad audience is to create a package. Packaging enforces the rigor of the code development and maintenance process, including documentation, testing, use, and accessibility. The goals of R packaging reinforce many of the ECO-PCX's goals and are therefore compatible with the overall USACE model review process.  
* Significance: The code documentation currently contains enough detail for use by intermediate+ R users, but packaging would help all levels of R users.  
* Resolution: Models developed on the R platform with the intention of being used widely throughout USACE should be packaged. Packaging R code and submitting to CRAN is the gold standard for quality and availability within the R community. However, packaging is challenging and time consuming. Authors should consider the cost/benefit of packaging and consider for the future if not during model review.  
* **Response: Concur.** The author agrees that an R package would be a fruitful outlet for vetting the model and sharing both internal and external to USACE. However, as the reviewer acknowledges, review by CRAN can be time-consuming, and model development has been conducted under limited project planning timelines. CRAN review and packing will be pursued for future versions of the model.  


*Comment 2: Format R code using an accepted style guide.*  

* Basis: Following standard practice of using a coding style guide improves readability and use of the model.  
* Significance: The code currently contains an adequate amount of code comments so that it is readable, but use of the R functions would benefit from applying a standard coding style. The model is usable in its current form by an intermediate R user, but would be more usable to beginner R users if this change were made.  
* Resolution: Follow the Google's R Style Guide. See the example below of the edited connectivity function that follows this style guide.  
* **Response: Concur.** Standardized style facilitates reuse of code and uptake by a user-community. The connectivity function was restructured based on a majority of the reviewer's suggestions. Some stylistic suggestions were not used due to the author's preference to maintain readability for a USACE user audience.   


*Comment 3: Organize "user manual" in the form of an R vignette.*  

* Basis: A standard convention within the R community is to organize "user manual" information in the form of a "vignette".  
* Significance: The R vignette style has proven to be one of the most effective methods for getting users quickly up-to-speed using a new R package. Vignettes introduce concepts and show the user exactly how to use the code including working code examples. Examples progress from simple to gradually complex to help the user develop
confidence with the new package.  
* Resolution: The code examples provided in the Evaluation and Application sections of the model documentation could easily be converted into the typical R vignette style.  
* **Response: Non-concur.** The reviewer is correct that vignettes are a standard convention within the R community. However, a USACE audience is target user community for the WUCT. As such, a standardized model development procedure following Grant and Swannack (2008) was applied here: conceptualize, quantify, evaluate, and apply. As mentioned above, CRAN formatting will be used in the future, if the model is refined and a package is pursued.  


*Comment 4: Add data management functions.*  

* Basis: Although the connectivity function makes the total accessible habitat calculation, the author admits: "While manual entry of model inputs may be sufficient for small watersheds, data management becomes challenging as watershed size and the number of alternatives increases. As an example of this increasing problem size, a standardized data structure was developed, which can flexibly manage larger numerical problems." Therefore it would be helpful to refactor the code in the Application section into a series of functions that accept the standardized input that was defined and reshape the data into the format required by the connectivity function. Functions should be created that reshape the model output into a more readable format as done in the example.  
* Significance: Refactoring the project example code into data management functions would greatly ease the adoption of the model by USACE staff. Although these helper functions are not strictly needed as demonstrated in the examples, their creation would greatly standardize the best practice application of the model. If the model author needed to perform these data management functions, other users of the model will certainly need to as well. Why not encapsulate these best practice steps into functions? This would greatly strengthen the model API. Toolkits have multiple functions, currently the WUCT only has one function. Adding data management functions would put the "T" in WUCT!.  
* Resolution: The author has already prototyped the workflow and identified the process improvement steps in the example. Refactoring the example code into functions should not be a very time consuming process.  
* **Response: Non-concur.** Two example applications have been provided to facilitate application. However, these are two examples of dozens of potential uses of the connectivity function. Development of generic data management functions would be challenging, given the diversity of potential uses of the function. For instance, complete enumeration of alternatives may be possible in a small basin with less than 20 barriers (e.g., the Bronx River application). Conversely, optimization algorithms may be required for large watersheds with hundreds of barriers (e.g., Neeson et al. 2015). As such, there is not a typical model application, and user-driven data management is likely to be needed (as is the case with many models).  


*Comment 5: Define "adjacency matrix" and how to construct one.*  

* Basis: The adjacency matrix is a key input to the connectivity function. Ensuring that users understand exactly what the adjacency matrix is, how it works, and how to construct one is key to successfully applying the WUCT.  
* Significance: Using Figure 4, examples, and with a close inspection of the connectivity function code, an intermediate R user can figure out what an adjacency matrix is and how to construct one. However, wide adoption of this model will be limited if this concept is not clearly explained.  
* Resolution: The model vignette should include a definition of an adjacency matrix and an explanation of how to build one. The beginnings of an explanation are included in Figure 4. Augment that schematic with a couple more examples of different network topologies and the ways to represent those in an adjacency matrix.  
* **Response: Concur.** Additional description has been added on the construction of an adjacency matrix.  


*Comment 6: Define "total accessible habitat".*  

* Basis: The primary metric that this model produces is the total accessible habitat value. However, the documentation only contains the following description of the metric: The outcome of the WUCT is a metric of total accessible habitat, which incorporates connectivity, quantity, and quality. Since this is not a widely understood metric, providing a definition of this will help users understand how to interpret it.  
* Significance: Although the name of the model provides some context and the name of the metric has high face validity, it would be helpful to provide more of a discussion of the metric. To help describe what the metric is measuring, it would helpful to provide a discussion of the effects of differing connectivity, quantity, and quality on the range of expected values.  
* Resolution: Including a discussion of the metric could be illustrated with several examples that demonstrate the range of outputs to be expected.  
* **Response: Concur.** The discussion of the metric was enhanced.  


*Comment 7: Add a graphing functions.*  

* Basis: Since the nature of this model is so graphical, it would be highly beneficial to include standard graphing functions to display model inputs and outputs.  
* Significance: In the model code's current state, inputs and outputs are difficult to visualize, let alone display together in a single view. Network diagrams like Figures 1, 2, and 4 are powerful ways to visualize this type of data that integrates the model inputs and outputs into a single figure. Many excellent flow diagram types are available (i.e., network diagrams, Sankey diagrams, dendrograms) from several packages (e.g., igraph, ggnet2, networkD3). See r-graph-gallery or Network visualization with R for alternatives.  
* Resolution: Using existing flow diagram packages allows quick and flexible implementation of robust model output visualization options. See example below using the igraph package that was developed for model evaluation.  
* **Response: Non-concur.** The author does not think graphing functions are necessary to construct within the WUCT. This conclusion is largely driven by the reviewer's comment that "existing flow diagram packages allow quick and flexible implementation of robust model output visualization options." Becuase these algorithms are readily available, they do not need to be duplicated within the WUCT. Language was, however, added to the model quantification section, which points users to these tools.    


*Comment 8: Add more rigorous tests.*  

* Basis: Hadley Wickham makes a compelling argument for developing automated tests to ensure the quality of code in the R Packages, Testing chapter. Since model evaluation is a key component of the USACE model review process, it seems that developing a set of automated tests for model functions would be extremely beneficial.  
* Significance: Given the difficulty of visualizing the inputs and outputs of this model, automated test sets would provide a way of demonstrating that the model is doing what it is supposed to be doing.  
* Resolution: See the example below which applies best practice tools for the creation of a set of tests.  
* **Response: Non-concur.** Automated function testing is a powerful mechanism for trouble-shooting and error-checking code. Unfortunately, project planning timelines prohibited the development of automated functions for testing within the WUCT. The author feels that the variety of manual test cases adequately probes the potential solution space, which resulted in no known errors. This improvement may be pursued in the future as code develops toward an R package.  


#### **Reviewer 2: Clayton Ridenour (USACE Omaha District)**


The author is grateful for the reviewer's insights and feedback. Comments center largely on model parameterization to capture key biological features or processes. The WUCT was designed as a generalizable framework for a variety of taxa, which makes a comprehensive review of parameterization extremely challenging. The author has augmented the report per the reviewer's suggestions, but a comprehensive review of parameterization is beyond the scope of this document. 

*Comment 1: Key assumptions not well organized and explicitly communicated.*  

* Basis: Reader is unsure what uncertainties were considered in development of the framework, and accounted for as assumptions. Seems there are opportunities to better tie assumptions to limitations/concerns of application of model. For example, an assumption that could be directly addressed is the organism's motive to migrate, which is indirectly addressed with the habitat quality discussion, but is not directly discussed in light of model application. In this example, motive to migrate could have bearing on how model output is interpreted because a judgement on timing of migration (e.g., spring vs anytime) could be useful in planning decisions.  
* Significance: Low.  
* Resolution: Explicitly address key assumptions, either in their own sub-header section or individually throughout the document as appropriate or as conveniently related to an issue or uncertainty.  
* **Response: Concur.** The generality of the WUCT makes a comprehensive listing of assumptions challenging to compile due to the multitude of ways the model may be parameterized by users. Key scoping issues are currently addressed in the background section, and assumptions and limitations of the model platform are documented in the summary section. Issues were called out as bullet points to draw attention, but bold font has been added to further reinforce this information.  


*Comment 2: A clear purpose statement would help to establish better boundaries of use and expectations, aid in integrating assumptions (e.g., behavioral, hydrologic/water availability, flow regulation, etc.), and interpreting model output.*  

* Basis: The Overview section provides a general sense of how the model operates and what kind of inputs, but the reader is not sure yet "why" until the final sentence (which is more of a policy/procedural purpose). However, we recognize the value of keeping this modeling framework adaptable and why providing purpose/sideboards that are too specific could work against the intent of the flexible framework.  
* Significance: Low.  
* Resolution: Consider something like the following as the opening sentence of Overview section: "The purpose of the WUCT is to assess watershed scale habitat connectivity and quality for migratory organisms to inform restoration planning".  
* **Response: Concur.** The proposed sentence was added to the background section.  


*Comment 3: This model document does not discuss the consideration of the cost/benefit of invasive species.*  

* Basis: Risk of facilitating spread of invasive species should be a consideration in river connectivity studies.  
* Significance: Low.  
* Resolution: We are assuming/confident that the authors and many previous reviewers have carefully considered this concern (why we rate significance low here), so to minimally resolve, simply document it to provide the reader with suggestions for additional reading (e.g., similar to the bulleted list in Development Process section) on how invasive species considerations were or could be considered in this model. To further and more directly guide users of this model, consider discussing if/how the model could handle benefits and costs relative to the risk of invasive species. An example of an option to consider is to encourage users to iterate model runs with an invasive species as the focal organism, as this could at least provide contrast for decision making. If a barrier is effectively restricting spread of invasive species, could this "benefit" be realized somehow in the model output? Another operational consideration involves timing of migratory behavior. If the target native species and an invasive species do not migrate at the same time, and the permeability of the barrier were adjustable, (e.g., block access to fish ladder during the migratory period of invasive species), could the model realize such as a benefit? This example highlights our previous comment on addressing assumptions, and specifically on organism's motive to migrate.  
* **Response: Concur.** The review is correct in noting the potential application of the WUCT to invasive species issues. This additional application has been explicitly added to the scoping issues reviewed in the Background section.  


*Comment 4: Guidance on estimating passability.*  

* Basis: Because passability is a fundamental component of the WUCT, it is likely to receive significant scrutiny in the USACE planning study process (e.g., public and expert technical review/comment).  
* Significance: Moderate.  
* Resolution: Consider providing more discussion on available tools and methods to estimate passability; perhaps provide a list or appendix of common/approved models or estimators or approaches (e.g., expert panel or expert survey?).  
* **Response: Concur.** As noted, passability is perhaps the most crucial user-input to the WUCT, and passability is likely to be (and should be) scrutinized for each application of the model. Rather than provide a comprehensive listing of potential sources of passability data, this document attempts to meet the spirit of this comment in three ways: (1) citations have been added to the conceptualization section pointing readers to thorough reviews of passability assessment, (2) the example applications presented in Figure 3 apply different methods for quantifying passability, and (3) the Bronx River example application presents an alternative technique.  


*Comment 5: Guidance on estimating passability for multiple target species, and/or multiple guilds.*  

* Basis: Not clear how the user should implement the model in a case where dealing with two or more species with very different swimming ability or capacity to navigate a partial passability barrier. For example, a sturgeon (generally benthic and not good at navigating rock ramp), a darter (generally benthic but good with rock ramps; although may not be motivated to migrate?), and an Asian carp (generally pelagic but can likely navigate rock ramps). Assuming reasonably defensible estimates for passability were derived for each, how would the model deal with this range of input parameters to arrive at a single "connectivity" score in the model output?  
* Significance: Moderate.  
* Resolution: Provide clarification of intended use of model with multiple species that have very different passability estimates. For example, users should strive to combine all passability estimates into a single representative value to input into the model; users should run the model separately for each species/guild/community, then deal with synthesizing and interpreting post model-run period; other?  
* **Response: Concur.** Text was added to the Background section to clarify application in the context of multiple focal taxa.  


*Comment 6: User platform requires programing language.*  

* Basis: May be relatively few potential users that have experience/background using R language, and may deter use. However, the basic framework is relatively simple, and with the example code provided, we anticipate that most potential users could follow along and make adjustments where necessary.  
* Significance: Very low.  
* Resolution: The authors recognized this limitation and addressed it by offering line-by-line code for the example provided, and further offered to provide support and background on R programming; not sure much more can be done. Reviewers wanted to include this in the comments log, no action anticipated to close the comment
* **Response: Concur.** The lack of a user interface does limit the potential user community. Per reviewer-1's suggestion, the following text was added to clarify this limitation. "The current script-based format of the model requires familiarity with R. As such, the model is recommended for use by intermediate-level R users. Future versions will pursue a GUI to facilitate use by non-R users."

