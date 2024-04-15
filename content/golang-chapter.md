---
title: Golang chapter
---
# Why I want to do it

* I want to stabilize and streamline Go development in Omio
	* Semantic versioning of packages and [strong compatibility defined by the Go team](https://go.dev/doc/go1compat)
	* Improve gomio-pipelines
		* Split from gomio package (or create a go.mod and version it separately as done in x/tools)
		* Add Omio static checks (this is even more strict than other language stacks). I already created
	* Adding stuff to the GOmio should be decided by the community and not only partnerships
		* Some people in monetization use GO and they have their own packages
	* Leverage existing GO tooling
		* PProf
		* Make other communities jealous 😁
* I want to ensure good standard around the architecture of the code and best practices
	* But I don't think that architecture hegemony is necessary. Tooling hegemony is enough.
* Forcing people update the dependencies, go versions
	* Need probably push from somebody on higher levels. In case of Java community Petr Hajek does it.
	* Dependabot - it should be easier because Go community respects backward compatibility a lot

* Go talks!
	* Learn together, brainstorm, easy personal goals for junior, medior developers
	* Create Omio Golang stickers and give them to people who did talks 😄
	* Possible talks from my side
		* Static analysis in GO! - Writing your first analyzer
		* Leveraging go tools to your benefit
			* x/tools
			* Multiple go modules in one repository