---
date: 2026-08-15
tags:
  - crawling
  - python
  - selenium
  - patent
  - automation
---
`main` 함수에서 `input` 받아서 크롤링 실행할 것이다.

`Crawler` 클래스에 크롤링 로직 함수 추가할 것이다.

```python
# config.py

# 테스트용 출원번호
TEST_APPLNOS = [
    '1020110009566',
    '1020110009567', 
    '1020110009568',
    '1020110009569',
    '1020110009570'
]

# 출력 설정
OUTPUT_CONFIG = {
    'auto_save': True,  # 자동 저장 여부
    'save_format': 'json',  # 저장 형식 (json, csv)
    'include_timestamp': True,  # 파일명에 타임스탬프 포함
    'output_dir': 'results'  # 출력 디렉토리
}
```

```python
class Crawler:
	# ...
	def crawl_single_patent(self, applno: str) -> Optional[Dict]:
        """단일 특허 데이터 크롤링"""
        try:
            logger.info(f"특허 데이터 크롤링 시작: {applno}")
            
            # XMLHttpRequest 오버라이드하여 응답 캡처
            self.driver.execute_script("""
                window.capturedData = null;
                const originalXHRSend = XMLHttpRequest.prototype.send;
                const originalXHROpen = XMLHttpRequest.prototype.open;
                
                XMLHttpRequest.prototype.open = function(method, url, ...args) {
                    this._method = method;
                    this._url = url;
                    return originalXHROpen.call(this, method, url, ...args);
                };
                
                XMLHttpRequest.prototype.send = function(data) {
                    if (this._url && this._url.includes('/dynaPath/')) {
                        console.log('dynaPath 요청 감지:', this._url);
                        
                        // 응답 캡처를 위한 이벤트 리스너 추가
                        this.addEventListener('load', function() {
                            if (this.status === 200) {
                                console.log('응답 캡처 성공');
                                try {
                                    window.capturedData = JSON.parse(this.responseText);
                                } catch (e) {
                                    window.capturedData = { error: 'JSON parse error', response: this.responseText };
                                }
                            } else {
                                window.capturedData = { error: 'HTTP error', status: this.status, response: this.responseText };
                            }
                        });
                        
                        this.addEventListener('error', function() {
                            window.capturedData = { error: 'Network error' };
                        });
                    }
                    return originalXHRSend.call(this, data);
                };
            """)
            
            # openDetail 호출
            self.driver.execute_script(f"openDetail('kpat', '{applno}', '', null)")
            time.sleep(5)
            
            # 캡처된 데이터 확인
            captured_data = self.driver.execute_script("return window.capturedData")
            
            if captured_data:
                logger.info(f"✅ {applno} 크롤링 성공!")
                return captured_data
            else:
                logger.error(f"❌ {applno} 크롤링 실패")
                return None
                
        except Exception as e:
            logger.error(f"크롤링 중 오류 발생 ({applno}): {e}")
            return None
    
    def crawl_multiple_patents(self, applno_list: List[str]) -> Dict[str, Optional[Dict]]:
        """여러 특허 데이터 크롤링"""
        results = {}
        successful_count = 0
        failed_count = 0
        
        try:
            # 드라이버 초기화
            self.setup_driver()
            
            # 사이트 방문
            logger.info("KIPRIS 사이트 방문")
            self.driver.get('https://www.kipris.or.kr/khome/search/searchResult.do')
            time.sleep(10)
            
            # JavaScript 객체 확인
            logger.info("JavaScript 객체 확인")
            dp_exists = self.driver.execute_script("return typeof window.dp === 'object'")
            openDetail_exists = self.driver.execute_script("return typeof window.openDetail === 'function'")
            
            if not (dp_exists and openDetail_exists):
                logger.error("❌ 필요한 JavaScript 객체가 없음")
                return results
            
            logger.info("✅ 필요한 객체들이 존재함!")
            
            # 각 출원번호 처리
            for i, applno in enumerate(applno_list, 1):
                logger.info(f"\n=== [{i}/{len(applno_list)}] 출원번호 {applno} 처리 ===")
                
                result = self.crawl_single_patent(applno)
                results[applno] = result
                
                if result:
                    successful_count += 1
                else:
                    failed_count += 1
                
                # 요청 간 대기 (서버 부하 방지)
                if i < len(applno_list):
                    logger.info(f"다음 요청을 위해 {self.request_delay}초 대기...")
                    time.sleep(self.request_delay)
            
            logger.info(f"\n=== 크롤링 완료 ===")
            logger.info(f"총 처리: {len(applno_list)}개")
            logger.info(f"성공: {successful_count}개")
            logger.info(f"실패: {failed_count}개")
            
        except Exception as e:
            logger.error(f"배치 크롤링 중 오류: {e}")
        finally:
            self.close()
        
        return results
    
    def save_results(self, results: Dict[str, Optional[Dict]], filename: str = None):
        """결과를 JSON 파일로 저장"""
        if filename is None:
            timestamp = time.strftime("%Y%m%d_%H%M%S")
            filename = f"kipris_results_{timestamp}.json"
            
        # 출력 디렉토리 생성
        output_dir = OUTPUT_CONFIG.get('output_dir', 'results')
        if not os.path.exists(output_dir):
            os.makedirs(output_dir)
            logger.info(f"출력 디렉토리 생성: {output_dir}")
        
        # 전체 경로 생성
        full_path = os.path.join(output_dir, filename)
        
        try:
            with open(full_path, 'w', encoding='utf-8') as f:
                json.dump(results, f, indent=2, ensure_ascii=False)
            logger.info(f"결과가 {full_path}에 저장되었습니다.")
        except Exception as e:
            logger.error(f"결과 저장 실패: {e}")
    
    def close(self):
        """드라이버 종료"""
        if self.driver:
            self.driver.quit()
            logger.info("드라이버 종료 완료")
```



```python
def main():
    """메인 함수"""
    logger.info("=== KIPRIS 특허 데이터 크롤링 도구 시작 ===")
    
    # 테스트할 출원번호 목록 (config.py에서 가져옴)
    test_applnos = TEST_APPLNOS
    
    # 크롤러 초기화
    crawler = Crawler()
    
    try:
        print("1. 단일 출원번호 크롤링")
        print("2. 여러 출원번호 배치 크롤링")
        print("3. 사용자 정의 출원번호 크롤링")
        choice = input("선택하세요 (1, 2, 또는 3): ")
        
        if choice == "1":
            # 단일 출원번호 크롤링
            applno = input("출원번호를 입력하세요: ").strip()
            if applno:
                crawler.setup_driver()
                crawler.driver.get('https://www.kipris.or.kr/khome/search/searchResult.do')
                time.sleep(10)
                
                result = crawler.crawl_single_patent(applno)
                if result:
                    print("\n=== 크롤링 결과 ===")
                    print(json.dumps(result, indent=2, ensure_ascii=False))
                    
                    # JSON 파일로 저장
                    timestamp = time.strftime("%Y%m%d_%H%M%S")
                    filename = f"kipris_data_{applno}_{timestamp}.json"
                    
                    # 출력 디렉토리 생성
                    output_dir = OUTPUT_CONFIG.get('output_dir', 'results')
                    if not os.path.exists(output_dir):
                        os.makedirs(output_dir)
                    
                    full_path = os.path.join(output_dir, filename)
                    with open(full_path, 'w', encoding='utf-8') as f:
                        json.dump(result, f, indent=2, ensure_ascii=False)
                    logger.info(f"결과가 {full_path}에 저장되었습니다.")
                else:
                    print("크롤링 실패")
        
        elif choice == "2":
            # 여러 출원번호 배치 크롤링
            logger.info(f"배치 크롤링 대상: {len(test_applnos)}개 출원번호")
            results = crawler.crawl_multiple_patents(test_applnos)
            
            # 결과 출력
            print("\n=== 크롤링 결과 요약 ===")
            for applno, result in results.items():
                print(f"\n출원번호: {applno}")
                if result and isinstance(result, dict) and 'error' not in result:
                    print("✅ 성공")
                    print(f"데이터 키: {list(result.keys())}")
                else:
                    print("❌ 실패")
                    if result:
                        print(f"오류: {result}")
            
            # 결과 저장
            crawler.save_results(results)
        
        elif choice == "3":
            # 사용자 정의 출원번호 크롤링
            print("출원번호를 쉼표로 구분하여 입력하세요 (예: 1020110009566,1020110009567)")
            user_input = input("출원번호: ").strip()
            
            if user_input:
                applno_list = [applno.strip() for applno in user_input.split(',') if applno.strip()]
                logger.info(f"사용자 정의 크롤링 대상: {len(applno_list)}개 출원번호")
                
                results = crawler.crawl_multiple_patents(applno_list)
                
                # 결과 출력
                print("\n=== 크롤링 결과 요약 ===")
                for applno, result in results.items():
                    print(f"\n출원번호: {applno}")
                    if result and isinstance(result, dict) and 'error' not in result:
                        print("✅ 성공")
                        print(f"데이터 키: {list(result.keys())}")
                    else:
                        print("❌ 실패")
                        if result:
                            print(f"오류: {result}")
                
                # 결과 저장
                crawler.save_results(results)
            else:
                print("출원번호가 입력되지 않았습니다.")
        
        else:
            print("잘못된 선택입니다.")
            
    except Exception as e:
        logger.error(f"메인 실행 중 오류: {e}")
    finally:
        crawler.close()
        logger.info("=== 크롤링 도구 종료 ===")
```