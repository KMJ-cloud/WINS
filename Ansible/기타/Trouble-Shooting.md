## 플레이북 작성 관련
   1. yaml 붙이기 안됨
        - yaml 문법상 같은 파일 안에 --- 로 나누게 되면 다른 파일로 인식 함  
          - 예) 쿠버네티스 yaml

            <!--![image](https://github.com/KMJ-cloud/WINS/assets/71053769/2e2e89fb-3ad3-4cc0-861e-654675712a9a)-->
            ![image](https://github.com/KMJ-cloud/WINS/assets/71053769/e56b49fc-8e0e-4522-a150-2263cc5b309e)

          - Ansible Playbook 에서는 --- 을 문법 에러로 인식
                        
            ![image](https://github.com/KMJ-cloud/WINS/assets/71053769/a86bd143-2fcb-4bf6-bfa7-c91af1256b48)


        - 해결
          - playbook 명령어 실행 시, 여러 yaml 지정하여 배포  
            - ![image](https://github.com/KMJ-cloud/WINS/assets/71053769/79674b63-6d09-43a0-8119-1acc541ff311)
          - 결과 같이 출력
            - ![image](https://github.com/KMJ-cloud/WINS/assets/71053769/e9eabb4d-8b26-44a1-b0eb-0c1c1d85d6c9)
          
   2. sshd conf 설정 시, 멱등성 보장 실패 (정규표현식으로 해결)
   3. rc코드 0 으로 받아야 정상 작동
## ansible 팁
   1. ansible 로그 (ok, changed) -> 앤서블 팁 이런쪽으로 빼기
