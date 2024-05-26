Cybersecurity Architech 

Role and Mindset 
- Talking to the stakeholders involved in the systems 
- Looking at the areas of the blueprint 
- Defining the Domains 
	- User 
	- Edge Devices 
	- Network(s)
	- Data Infastructure 
	- Data Resources 

Context to focus and look at 
- Bus Context 
	- How information moves around 
	- who is involved 
- System Context 
	- What application are communcating 
	- How does it communicate does it go through a network of systems 	
- Cybersecurity Arch Overview
	- Workflows and Systems 
	- Reporting 
	- Alert Systems 
	- Controls and reviews 

A Cybersecurity LifeCycle 
	1. Risk Analysis 
	2. Policy 
	3. Arch Design 
	4. Implement 
	5. Management (Admin)
	6. Audits / Assesment 

A Frame work View (NIST-CS)
	1. Indentify
	2. Protect
	3. Detect 
	4. Responses
	5. Recover


Domain of Operations 
 - Domains
	1. User (IAM)(Arch)
		- Federation Access Management
			- Link IAM to External IAM
		- User Groups 
			- Humans 
			- Non-Human
		- User Sub Groups 
                        - Humans 
			- Non-Human
		- Id access Rights 
		- Access rights (Systems)
		- Information Store (Directory)
			- DB 
			- Mapping Schema 
			- Protocal (e.g, LDAP)

		- Role Management 
			- User Roles 
			- User Role Deffinition 
			- Mapping Access Rights
			- Who approves Role Appointment
		- Authenticating the User 
			- Something you Know (Password) 
			- Something you have (2FA)
			- Something you Are  (Biometric)
			- MFA (All Three or More)
			- Passwordless (SSO)
		- Privillage Access Managment(PAM) (Kingdom Keys)
			- Deploying MFA 
			- Session and Activity Recording 
		- Audits and Monitors 
			- User Behaver Analytics (UBA)
			- User Entity Behavier Analytics (UEBA)
	2. EndPoint(Devices)
		- All Devices Used by the Users and Entity
			- Desktop 
			- Laptops 
			- Mobile Phones 
			- Talets 
			- IoTs
			- other Devices 
		- Busniess Devices and Personal Devices
			- BYO(D,IT,Cloud)
				- Well define Program 
				- Poorly define Program 
			- Actions and Controls 
				- Require Consent upfront 
				- Software Approvals 
				- Hardware Approvals
					- Level of Hward Config
					- Level of Hward type 
				- Services Approvals 
		- Holistic Endpoint Management (HEMs)
			- Policies 
			- Patch Controls 
			- Info Alerts 
			- Controls 
				- Inventory Management 
				- Security Policies 
				- Patching Policies 
				- Encryption Policies
				- Remote Wipe Policies 
				- Device Location Tracking Policies
				- Anti-Virus and EDR
				- Disposal 

		- Software
			- OS Entities
			- Firmwares
			- General Software 
 
	3. Networking 
		- Firewalls
			- Seperation in mind
			- Filering or Packet Analysis 
			- Managing of In/out Traffic 
			- Traffic Controls 
			- Types of Firewalls
				- Packet Filtering 
				- Stateful Packet Inspection 
				- Proxy 
				- Netword Address Translation(NAT)
		- Segment 
			- Tri-Home 
			- Basic DMZ
				- Firewall Based
				- Multiple Firewalls 
			- Multi-Tiered 
				- 1st Tier Firewall
				- 2nd Tier Firewall
				- 3rd Tier Firewall
			- Micro-Segmentation
		- VPN
			- Secure channel of a untrusted network
			- Information Encyption 
			- Secure lines or Pipes 
			- limited Inspection 
			- VPN Technology 
				- SSH (App Layer)
				- TLS (Transport Layer)
				- IPSEC (Network)
				- PPTP/L2TP (Data link)
			- Network Base 
				- Simple
				- Catches All
			- Application Specific
				- Granularity 
				- Control 
		- Secure Access Service Edge (SASE)
			- is Network Security and WAN over the Cloud
			- SD-WAN
			- Cloud Mixing
			- ID Control
		- Other Nework Security Areas
			- 4G(LTE), 5G, Satellite 
			- Fiber and Submarine 
			- Wireless 
			- Microwave and Spectrum 
			
	4. Application 
		- All software have bugs and Vulnarability 
		- Software Development Life Cycle
		- Secure Coding
			- Secure Coding Practice
				- OWASP.org 
			- Trusted Libaries 
			- Standard Arch
			- Mistakes to Avoid 
				- OWASP-Top10
			- Software Bill of Materials 
				- Components 
				- Libs
				- Dependies 
				- Versions 
				- Origins 
				- Vulunarbilities 

		- Vulnarability Testing 
			- Static Application Security Testing(SAST)
				- White Box Testing
				- Source code is analysised 
				- Looking for Vulnarable elements 
				- Find Vulnabilities Earler

			- Dynamic Appliaction Security Testing(DAST)
				- Black Box Testing 
				- Running the App and checking 
				- Find Vulnarabilities Later

                        - ChatBot Code Checking 
				- Debug Code 
				- Code Analysis
				- Inject Vulnarable Codes 
				- Exposer of Interlectual Property

		- Adopting DevSEC-OPS Practices
		- Security In Phases 
		- Security by Design
	5. Data
		- Governance
			- What is defined 
				- Policy 
				- Classification
				- Catalog 
				- Resilience
		- Discovery 
			- Data Base 
			- Files 
			- Store Location for Structured 
			- Store location for UnStructured
			- DLP
			- Networking
 
		- Protection
			- ecryption
				- In rest 
				- In motion 
				- In store
			- Key Managment 
			- Access Control 
			- Backups 

		- Compliance to Regulation 
			- Reporting 
			- Retention 
		- Detection 
			- User behavior Analytics (UBA) 
			- Monitoring 
			- Alerts 
		- Responses 
			- Cases 
			- Dynamic PB
			- Orchistration 
			- Auth
		- Top by Five things to Reduce Attacks
			- AI 
			- DevSecOps 
			- Incident Response 
			- Cyptography
			- Training Employees
 

 - Support
	- Security Operation Center(SOC)
		- Detection (SIEM)
			- Monitor
			- Analyse
				- Define Rules
				- Anomalies 
			- Report 
				- Logs 
				- Events 
				- Trends
			- Hunt Threats
				- Early Detection 
				- Using XDR and SIEMs

		- Detection (ERD/XDR - Extended Detection Response)
			- Edge Detection
			- Fedurated Search 
		- A good combination is SIEM and XDR
		- Response
			- Attack map
				- Recon	
				- Attack
				- Persistance
			
			- Resource 
				- Humans with Knowledge 
			- Triage
			- Remediate 
			- Using Security Orchestration Automation and Response
				- Automated Orchestration
				- SOAR
					- Case 
						- Information 
						- Artifacts 
						- Tracking
						- Investigation
					- Dynamic Play Books
						- Runing the Remidiations
					- Automation and Orchestration 
						- Only Automate preevents
						- Breach Notifications
							- what data impacted 
							- where impact from 
							- Regulatory Requirements
		
 



 

