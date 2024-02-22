## 플레이북 작성 관련
   1. yaml 붙이기 안됨
        - yaml 문법상 같은 파일 안에 --- 로 나누게 되면 다른 파일로 인식 함  
          - 예) 쿠버네티스 yaml
            ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: deploy-account
              #namespace: default
              labels:
                project: example
            spec:
              replicas: 2
              selector:
                matchLabels:
                  app: account
              template:
                metadata:
                  labels:
                    app: account
                    project: example
                spec:
                  containers:
                  - image: hsj8314/account:v1
                    name: account
                    ports:
                    - containerPort: 5000
                      protocol: TCP
            
            --- 
            apiVersion: v1
            kind: Service
            metadata:
              name: service-account
              #  namespace: default
              labels:
                project: example
            spec:
              type: ClusterIP
              ports:
              - port: 80
                protocol: TCP
                targetPort: 5000
              selector:
                app: account
                project: example
            ```
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
      - 변경 전
         - 플레이북 내용
            ```yaml
            - name: STEP 3-3. Comment PasswordAuthentication if it's set to no # PasswordAuthentication no 주석 처리
               replace:
                 path: /etc/ssh/sshd_config # 대상 파일
                 regexp: 'PasswordAuthentication no' # 대상 문자열
                 replace: '#PasswordAuthentication no' # 변경될 문자열
               when: grep_result.rc == 0
            ```
         - 결과
           - 실행한 만큼 '#' 문자가 계속 생김 (멱등성 보장 X)
             
             ``` #####PasswordAuthentication no```
      - 표준정규표현식으로 변경 후
         - 플레이북 내용        
            ```yaml
               - name: STEP 3-3. Comment PasswordAuthentication if it's set to no # PasswordAuthentication no 주석 처리
                  replace:
                    path: /etc/ssh/sshd_config # 대상 파일
                    regexp: '^\s*PasswordAuthentication\s+no' # 대상 문자열
                    replace: '#PasswordAuthentication no' # 변경될 문자열
                  when: grep_result.rc == 0
            ```
         - 결과
           - 몇 번을 실행해도 '#' 안생김 (멱등성 보장 O)
             
             ``` #####PasswordAuthentication no ```
   3. rc 값을 0 으로 받아야 정상 작동
      - 작업 수행 시 return code(rc)를 받아 옴
      - rc 값을 가지고 조건문을 수행 가능
      - return code 리스트  
         |rc 번호|내용|비고|
         |---|---|---|
         |0 |(SUCCESS): 작업이 성공적으로 완료됨.|ㅇ|
         |1 |(FAILURE): 작업이 실패함.|ㅇ|
         |2 |(UNREACHABLE): 대상 호스트에 연결할 수 없음.|ㅇ|
         |3 |(PARSE_ERROR): YAML 파일 구문 분석 오류.|ㅇ|
         |4 |(DEPENDENCY_FAILURE): 작업의 종속성이 실패함.|ㅇ|
         |5 |(COMMAND_TIMEOUT): 명령 실행이 시간 초과됨.|ㅇ|
         |250 |(RESCUE_MODE): 작업이 대체 모드로 전환됨.|ㅇ|
         |251 |(RETRY): 작업이 재시도됨.|ㅇ|
         |252 |(UNKNOWN_ERROR): 알 수 없는 오류.|ㅇ|
         |253 |(MISSING_ARGS): 필요한 인수 누락.|ㅇ|
         |254 |(PROJECT_NOT_FOUND): 프로젝트를 찾을 수 없음.|ㅇ|
         |255 |(EXCEPTION): 예외 상황 발생.|ㅇ|

## ansible 팁
   1. ansible TASK 별 실행 결과 분석 방법

         |TASK 실행 결과|내용|
         |---|---|
         |OK|작업이 성공적으로 수행되었고, 변경 사항이 없음을 나타냅니다.|
         |CHANGED|작업이 성공적으로 수행되었고, 목표 상태에 변경이 있었음을 나타냅니다. <br>예를 들어 파일을 복사하거나 서비스를 다시 시작하는 등의 경우에 발생할 수 있습니다.|
         |FAILED|작업이 실패했음을 나타냅니다. <br>예상치 못한 문제 또는 작업의 실패가 발생한 경우입니다.|
         |UNREACHABLE|대상 호스트에 접속할 수 없을 때 나타납니다. <br>네트워크 문제 또는 호스트가 다운된 경우 등이 해당합니다.|
         |SKIPPED|작업이 실행되지 않았음을 나타냅니다. <br>일반적으로 when 절이나 기타 조건에 의해 작업이 생략되었을 때 발생합니다.|
