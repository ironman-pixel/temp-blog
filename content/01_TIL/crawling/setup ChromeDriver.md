---
date: 2026-08-15
tags:
  - crawling
  - selenium
  - chromedriver
  - python
  - automation
---
## config.py 에 설정 추가
```python
"""
크롤링 설정 파일
"""

# 브라우저 설정
BROWSER_CONFIG = {
    'headless': True,  # True로 설정하면 브라우저 창이 보이지 않음
    'wait_timeout': 30,  # JavaScript 로딩 대기 시간 (초)
    'request_delay': 3,  # 요청 간 대기 시간 (초)
}

# 로깅 설정
LOGGING_CONFIG = {
    'level': 'INFO',  # DEBUG, INFO, WARNING, ERROR
    'format': '%(asctime)s - %(levelname)s - %(message)s'
}
```

## Crawler class 정의
```python 
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

from config import BROWSER_CONFIG


# 로깅 설정
logging.basicConfig(
	level=getattr(logger, LOGGING_CONFIG['level'], 
	format=LOGGING_CONFIG['format']
)
logger = logging.getLogger(__name__)


class Crawler:
	"""크롤링 클래스"""
	
	def __init__(self):
		"""
		초기화
		
		Args:
			headless: 헤드리스 모드 사용 여부 (None이면 config.py에서 가져옴)
		"""
		self.headless = BROWSER_CONFIG['headless']
		self.request_delay = BROWSER_CONFIG['request_delay']
		self.driver = None
	
	
	def setup_driver(self) -> webdriver.Chrome:
		"""Chrome 드라이버 설정"""
		
		# 브라우저 옵션 설정
		chrome_option = Options()
		
		if self.headless:
			chrome_options.add_argument('--headless')
		
		chrome_options.add_argument('--no-sandbox')
        chrome_options.add_argument('--disable-dev-shm-usage')
        chrome_options.add_argument('--disable-blink-features=AutomationControlled')
        chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
        chrome_options.add_experimental_option('useAutomationExtension', False)
        
        try:
            service = Service(ChromeDriverManager().install())
            self.driver = webdriver.Chrome(service=service, options=chrome_options)
            self.driver.execute_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined})")
            logger.info("Chrome 드라이버 초기화 완료")
            return self.driver
        except Exception as e:
            logger.error(f"드라이버 초기화 실패: {e}")
            raise
    
    
    def main():
	    """메인 함수"""
	    logger.info("=== 크롤링 도구 시작 ===")
	    
	    # 크롤러 초기화
	    crawler = Crawler()
	    
	    try: 
		    # TODO
		except Exception as e:
			logger.error(f"메인 실행 중 오류: {e}")
		finally:
			crawler.close()
			logger.info("=== 크롤링 도구 종료 ===")

if __name__ == "__main__":
	main()
```