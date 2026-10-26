# Notes from the Doc Merge

* I did not carry forward Zahed's definiton of dialog context. Context has a meaning in the industry and I didnt want to redefine it, so I just restated the definition of context itself. I also found Zahed's definition combined concepts that are just about context as it exists today, along with concepts unique to this protocol (identifiers and lifecycle state). Those are just properties of the agentic dialog in my definition

* A lot of the architecture text and layers are really just references to other protocols and their features. I trimmed this out and condensed it elsewhere

* I largely reused my terminology (agentic dialog) vs. the Zahed draft (dialog context), due to the reasons above. 

* I largely kept the protocol blocks from the Zahed draft, which were absent in mine, and then adjusted terminology

* Restructured text a bit so that the section sub-headings matched Zahed's architecture functions (e.g., there was no section on Exchange, even though it appeared in the diagram), and added in some missing text where it was thin in the doc (e.g., lifecycle)

* Multimodal negotiation was listed as a transport feature but it doesn't belong there; it is part of exchange of data in the agentic dialog management. I largely moved it. 

* Similar with progress signaling - this was in the transport section but is part of lifecycle management


OPEN ISSUES

* Interimediaries: I think this should not be included, and instead we should view ACP as a two-party protocol. Multiparty is handled by a combination of independent agentic dialogs; proxies are handled by back-to-back usage of the protocol. The only thing we need is to add some identity concepts to represent things like original princpal or different users contributing context. This is a lesson hard-learned from SIP. However for now I have kept this language in the doc

* High throughput reliable delivery - I think should be out of scope

* I don't think priority is needed in the protocol, nor is it a transport level consideration. We just need to allow multiple parallel agentic dialogs (and associated transport connections) between clients and servers. I think muxing is a bad idea here and not needed. There is no overhead anymore for many transport connections. I have however left it in the doc