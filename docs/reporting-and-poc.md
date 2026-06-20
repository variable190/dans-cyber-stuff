# Reporting & POC

## Report Sections

| Section                  | Description |
|--------------------------|-------------|
| Title                    | Concise vulnerability name including type, target, and impact |
| Severity                 | Overall severity rating (Low/Medium/High) |
| Impact                   | Impact rating (Low/Medium/High) |
| Exploitability           | Exploitability rating (Low/Medium/High) |
| CVSS Base Score          | Numerical CVSS base score |
| CVSS v3 Vector           | Full CVSS v3.1 vector string |
| Vulnerability Type       | Type of vulnerability (e.g. XSS, IDOR) |
| Target                   | Affected endpoint, parameter, domain, or asset |
| Impact                   | Detailed business and technical impact of exploitation |
| Description              | Clear explanation of the vulnerability cause and mechanics |
| Steps to Reproduce       | Concise, step-by-step reproduction instructions |
| Recommendation           | Specific remediation or mitigation steps |

### CVSS 3.1

#### Attack Vector

Shows how the vulnerability can be exploited.

- **Network (N)**: Attackers can only exploit this vulnerability through the network layer (remotely exploitable).
- **Adjacent (A)**: Attackers can exploit this vulnerability only if they reside in the same physical or logical network (secure VPN included).
- **Local (L)**: Attackers can exploit this vulnerability only by accessing the target system locally (e.g., keyboard, terminal, etc.) or remotely (e.g., SSH) or through user interaction.
- **Physical (P)**: Attackers can exploit this vulnerability through physical interaction/manipulation.

#### Attack Complexity

Depicts the conditions beyond the attackers' control and must be present to exploit the vulnerability successfully.

- **Low (L)**: No special preparations should take place to exploit the vulnerability successfully. The attackers can exploit the vulnerability repeatedly without any issue.
- **High (H)**: Special preparations and information gathering should take place to exploit the vulnerability successfully.

#### Privileges Required

Show the level of privileges the attacker must have to exploit the vulnerability successfully.

- **None (N)**: No special access related to settings or files is required to exploit the vulnerability successfully. The vulnerability can be exploited from an unauthorized perspective.
- **Low (L)**: Attackers should possess standard user privileges to exploit the vulnerability successfully. The exploitation in this case usually affects files and settings owned by a user or non-sensitive assets.
- **High (H)**: Attackers should possess admin-level privileges to exploit the vulnerability successfully. The exploitation in this case usually affects the entire vulnerable system.

#### User Interaction

Shows if attackers can successfully exploit the vulnerability on their own or user interaction is required.

- **None (N)**: Attackers can successfully exploit the vulnerability independently.
- **Required (R)**: A user should take some action before the attackers can successfully exploit the vulnerability.

#### Scope

Shows if successful exploitation of the vulnerability can affect components other than the affected one.

- **Unchanged (U)**: Successful exploitation of the vulnerability affects the vulnerable component or affects resources managed by the same security authority.
- **Changed (C)**: Successful exploitation of the vulnerability can affect components other than the affected one or resources beyond the scope of the affected component's security authority.

#### Confidentiality

Shows how much the vulnerable component's confidentiality is affected upon successfully exploiting the vulnerability. Confidentiality limits information access and disclosure to authorized users only and prevents unauthorized users from accessing information.

- **None (N)**: The confidentiality of the vulnerable component does not get impacted.
- **Low (L)**: The vulnerable component will experience some loss of confidentiality upon successful exploitation of the vulnerability. In this case, the attackers do not have control over what information is obtained.
- **High (H)**: The vulnerable component will experience total (or serious) loss of confidentiality upon successfully exploiting the vulnerability. In this case, the attackers have total (or some) control over what information is obtained.

#### Integrity

Shows how much the vulnerable component's integrity is affected upon successfully exploiting the vulnerability. Integrity refers to the trustworthiness and veracity of information.

- **None (N)**: The integrity of the vulnerable component does not get impacted.
- **Low (L)**: Attackers can modify data in a limited manner on the vulnerable component upon successfully exploiting the vulnerability. Attackers do not have control over the consequence of a modification, and the vulnerable component does not get seriously affected in this case.
- **High (H)**: Attackers can modify all or critical data on the vulnerable component upon successfully exploiting the vulnerability. Attackers have control over the consequence of a modification, and the vulnerable component will experience a total loss of integrity.

#### Availability

Shows how much the vulnerable component's availability is affected upon successfully exploiting the vulnerability. Availability refers to the accessibility of information resources in terms of network bandwidth, disk space, processor cycles, etc.

- **None (N)**: The availability of the vulnerable component does not get impacted.
- **Low (L)**: The vulnerable component will experience some loss of availability upon successfully exploiting the vulnerability. The attacker does not have complete control over the vulnerable component's availability and cannot deny the service to users, and performance is just reduced.
- **High (H)**: The vulnerable component will experience total (or severe) availability loss upon successfully exploiting the vulnerability. The attacker has complete (or significant) control over the vulnerable component's availability and can deny the service to users. Performance is significantly reduced.

