# Day 4 퀴즈 정답 및 해설

---

## 문제 1 정답: A) Virtual Private Cloud
**해설**: VPC는 Virtual Private Cloud의 약자로, AWS 클라우드에서 논리적으로 격리된 가상 네트워크를 의미합니다.

---

## 문제 2 정답: C) 65,536개
**해설**: CIDR 블록 10.0.0.0/16에서 /16은 앞의 16비트가 네트워크 부분이고, 나머지 16비트가 호스트 부분입니다. 따라서 2^16 = 65,536개의 IP 주소를 사용할 수 있습니다.

---

## 문제 3 정답: C) 외부에서 직접 접근할 수 없다
**해설**: 프라이빗 서브넷은 인터넷 게이트웨이로의 직접 라우팅이 없어 외부에서 직접 접근할 수 없습니다. 퍼블릭 IP도 자동 할당되지 않습니다.

---

## 문제 4 정답: B) 인터넷 게이트웨이로의 라우팅 경로
**해설**: 서브넷이 퍼블릭 서브넷이 되려면 라우팅 테이블에 인터넷 게이트웨이(0.0.0.0/0 → IGW)로의 경로가 있어야 합니다.

---

## 문제 5 정답: B) 프라이빗 서브넷의 아웃바운드 인터넷 접근
**해설**: NAT 게이트웨이는 프라이빗 서브넷의 리소스가 인터넷으로 아웃바운드 연결을 할 수 있게 해주지만, 인바운드 연결은 차단합니다.

---

## 문제 6 정답: B) 보안 그룹은 Stateful, NACL은 Stateless
**해설**: 보안 그룹은 연결 상태를 추적(Stateful)하여 응답 트래픽을 자동으로 허용하지만, NACL은 상태를 추적하지 않아(Stateless) 양방향 규칙이 모두 필요합니다.

---

## 문제 7 정답: A) Allow 규칙만 가능
**해설**: 보안 그룹은 기본적으로 모든 트래픽을 거부하고, Allow 규칙만 설정할 수 있습니다. Deny 규칙은 설정할 수 없습니다.

---

## 문제 8 정답: B) 규칙 번호 순서대로 평가하여 첫 번째 매칭 규칙 적용
**해설**: NACL은 규칙 번호 오름차순으로 평가하여 첫 번째로 매칭되는 규칙을 적용합니다. 따라서 규칙 순서가 중요합니다.

---

## 문제 9 정답: D) 1:1 연결만 가능
**해설**: VPC 피어링은 두 VPC 간의 1:1 직접 연결만 가능하며, 전이적 라우팅을 지원하지 않습니다. 또한 CIDR 블록이 중복되면 안 됩니다.

---

## 문제 10 정답: B) 중앙 집중식 연결 관리
**해설**: Transit Gateway는 여러 VPC, VPN, Direct Connect를 중앙에서 관리할 수 있는 네트워크 허브 역할을 하여 복잡한 네트워크 토폴로지를 단순화합니다.

---

## 문제 11 정답: B) 인터넷 기반 암호화 연결
**해설**: Site-to-Site VPN은 인터넷을 통한 IPSec 암호화 연결로, 빠른 구축이 가능하지만 인터넷 품질에 의존적입니다.

---

## 문제 12 정답: C) 일관된 네트워크 성능
**해설**: Direct Connect는 전용 물리적 연결을 통해 일관된 네트워크 성능과 낮은 지연시간을 제공합니다. 구축 시간이 오래 걸리고 비용이 높은 단점이 있습니다.

---

## 문제 13 정답: C) Longest Prefix Match (가장 구체적인 경로 우선)
**해설**: 라우팅 테이블에서는 가장 구체적인 경로(서브넷 마스크가 긴 경로)가 우선적으로 선택됩니다. 예를 들어 /24가 /16보다 우선합니다.

---

## 문제 14 정답: B) VPC CIDR의 두 번째 IP
**해설**: VPC에서 CIDR 블록의 두 번째 IP 주소(예: 10.0.0.2)는 DNS 서버용으로 예약되어 있습니다. 첫 번째는 네트워크 주소, 마지막은 브로드캐스트 주소입니다.

---

## 문제 15 정답: C) VPC당 하나만 연결 가능
**해설**: 인터넷 게이트웨이는 VPC당 하나만 연결할 수 있으며, 가용 영역에 관계없이 VPC 전체에 적용됩니다. 고가용성은 AWS에서 자동으로 관리합니다.

---

## 📊 점수 계산
- **90-100점**: 우수 - VPC 네트워킹 개념을 완벽히 이해했습니다
- **80-89점**: 양호 - 대부분의 VPC 개념을 잘 이해했습니다
- **70-79점**: 보통 - 기본 개념은 이해했으나 복습이 필요합니다
- **70점 미만**: 미흡 - VPC 기초 내용을 다시 복습해주세요

---

## 🔄 복습 권장 자료
점수가 낮은 영역에 대해 해당 강의 자료를 다시 확인해보세요:
- **VPC 개념**: [01-vpc-concepts.md](../day4/01-vpc-concepts.md)
- **VPC 생성**: [02-vpc-creation-subnets.md](../day4/02-vpc-creation-subnets.md)
- **라우팅/NAT**: [03-routing-nat.md](../day4/03-routing-nat.md)
- **VPC 연결**: [04-vpc-connectivity.md](../day4/04-vpc-connectivity.md)
- **하이브리드**: [05-hybrid-connectivity.md](../day4/05-hybrid-connectivity.md)