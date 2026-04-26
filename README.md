# TCT 실기 시험 Java Snippet 치트시트

> 기출문제 기반으로 실제 시험에 나올 수 있는 핵심 코드 패턴 정리

---

## 1. 파일 읽기 (가장 기본 — 거의 매번 출제)

### 1-1. 텍스트 파일 한 줄씩 읽기
```java
BufferedReader br = new BufferedReader(new FileReader("MONITORING.TXT"));
String line;
while ((line = br.readLine()) != null) {
    line = line.trim();
    if (line.isEmpty()) continue;
    // 처리 로직
}
br.close();
```

### 1-2. 구분자로 파싱 (# , | 탭 등)
```java
String[] parts = line.split("#");
String reqId     = parts[0];
String timestamp = parts[1];
String type      = parts[2];
String value     = parts[3];
```

### 1-3. CSV 파싱 (콤마 구분)
```java
String[] parts = line.split(",");
// 따옴표 제거가 필요한 경우
String cleaned = parts[0].replace("\"", "").trim();
```

---

## 2. JSON 처리 (Gson — SUB3/SUB4 단골)

### 2-1. JSON 파일 읽기
```java
import com.google.gson.*;

BufferedReader br = new BufferedReader(new FileReader("MODELS.JSON"));
StringBuilder sb = new StringBuilder();
String line;
while ((line = br.readLine()) != null) {
    sb.append(line);
}
br.close();

JsonObject json = JsonParser.parseString(sb.toString()).getAsJsonObject();
```

### 2-2. JSON Object 순회
```java
for (Map.Entry<String, JsonElement> entry : json.entrySet()) {
    String key = entry.getKey();                         // "resnet3.0"
    JsonArray arr = entry.getValue().getAsJsonArray();    // ["agent01","agent03"]
    
    List<String> list = new ArrayList<>();
    for (JsonElement e : arr) {
        list.add(e.getAsString());
    }
}
```

### 2-3. JSON 문자열 → 객체
```java
String jsonStr = "{\"name\":\"test\",\"value\":123}";
JsonObject obj = JsonParser.parseString(jsonStr).getAsJsonObject();

String name  = obj.get("name").getAsString();
int value    = obj.get("value").getAsInt();
long bigVal  = obj.get("bigValue").getAsLong();
double dVal  = obj.get("doubleVal").getAsDouble();
```

### 2-4. JSON 응답 생성
```java
JsonObject result = new JsonObject();
result.addProperty("correct", 8);
result.addProperty("total", 10);
result.addProperty("latency", 300);
// result.toString() → {"correct":8,"total":10,"latency":300}
```

### 2-5. JSON Array 생성
```java
JsonArray arr = new JsonArray();
arr.add("item1");
arr.add("item2");

JsonObject obj = new JsonObject();
obj.add("items", arr);
// {"items":["item1","item2"]}
```

### 2-6. 필드 존재 여부 체크
```java
if (obj.has("latency")) {
    long latency = obj.get("latency").getAsLong();
}
```

---

## 3. 콘솔 입/출력

### 3-1. 콘솔 입력 받기
```java
BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));
String input = stdin.readLine().trim();
```

### 3-2. 콘솔 출력 (정확한 포맷 필수!)
```java
System.out.println(matched + "/" + total);     // 8/10
System.out.println(String.format("%d/%d", matched, total));
```

> ⚠️ **주의**: 불필요한 로그 출력 금지! `System.out.println`으로 답만 출력

---

## 4. 자료구조 활용 패턴

### 4-1. HashMap — Key별 데이터 저장
```java
Map<String, String> pFlags = new HashMap<>();
Map<String, String> aFlags = new HashMap<>();

pFlags.put(reqId, value);  // 저장
pFlags.get(reqId);         // 조회
pFlags.containsKey(reqId); // 존재 체크
```

### 4-2. 그룹핑 (Key → List)
```java
Map<String, List<String>> groups = new HashMap<>();

// 방법1: computeIfAbsent (추천)
groups.computeIfAbsent(groupKey, k -> new ArrayList<>()).add(item);

// 방법2: 수동
if (!groups.containsKey(groupKey)) {
    groups.put(groupKey, new ArrayList<>());
}
groups.get(groupKey).add(item);
```

### 4-3. TreeMap — 정렬된 순서 유지
```java
TreeMap<String, List<String>> sorted = new TreeMap<>();
// Key 기준 자동 오름차순 정렬
```

### 4-4. Set — 중복 제거 / 빠른 포함 여부 체크
```java
Set<String> agentSet = new HashSet<>(agentList);
if (agentSet.contains(agentId)) {
    // ...
}
```

### 4-5. Map 순회
```java
for (Map.Entry<String, String> entry : map.entrySet()) {
    String key = entry.getKey();
    String val = entry.getValue();
}
```

---

## 5. Jetty HTTP 서버 (SUB3/SUB4 핵심!)

### 5-1. 기본 서버 세팅
```java
import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.handler.AbstractHandler;
import javax.servlet.http.*;
import java.io.*;

Server server = new Server(8080);
server.setHandler(new AbstractHandler() {
    @Override
    public void handle(String target, Request baseRequest,
            HttpServletRequest request, HttpServletResponse response)
            throws IOException {

        if ("/monitoring".equals(target) && "POST".equalsIgnoreCase(request.getMethod())) {
            handleMonitoring(request, response);
            baseRequest.setHandled(true);
        } else if ("/performance".equals(target) && "POST".equalsIgnoreCase(request.getMethod())) {
            handlePerformance(request, response);
            baseRequest.setHandled(true);
        }
    }
});
server.start();
server.join();  // 서버 종료 없이 대기 (문항3~4 요구사항)
```

### 5-2. HTTP 요청 Body 읽기
```java
private static String readBody(HttpServletRequest request) throws IOException {
    BufferedReader br = request.getReader();
    StringBuilder sb = new StringBuilder();
    String line;
    while ((line = br.readLine()) != null) {
        sb.append(line);
    }
    return sb.toString();
}

// 사용
JsonObject data = JsonParser.parseString(readBody(request)).getAsJsonObject();
```

### 5-3. HTTP 응답 보내기
```java
// 200 OK + 빈 응답
response.setStatus(200);

// 200 OK + JSON 응답
response.setStatus(200);
response.setContentType("application/json");
response.getWriter().write(result.toString());
```

### 5-4. 다중 엔드포인트 라우팅 패턴
```java
public void handle(String target, Request baseRequest,
        HttpServletRequest request, HttpServletResponse response) throws IOException {

    String method = request.getMethod();

    switch (target) {
        case "/monitoring":
            if ("POST".equalsIgnoreCase(method)) handleMonitoring(request, response);
            break;
        case "/performance":
            if ("POST".equalsIgnoreCase(method)) handlePerformance(request, response);
            break;
        case "/status":
            if ("GET".equalsIgnoreCase(method)) handleStatus(request, response);
            break;
    }
    baseRequest.setHandled(true);
}
```

### 5-5. Query Parameter 읽기 (GET 요청용)
```java
// GET /search?keyword=test&page=1
String keyword = request.getParameter("keyword");
String page = request.getParameter("page");
```

---

## 6. 시간/날짜 처리 패턴

### 6-1. 문자열 시간 범위 비교 (가장 빈출!)
```java
String timeWindow = "2025041010"; // yyyyMMddHH
String startTs = timeWindow + "0000"; // 20250410100000
String endTs   = timeWindow + "5959"; // 20250410105959

// 문자열 비교 (yyyyMMddHHmmss 는 사전순 = 시간순)
if (timestamp.compareTo(startTs) >= 0 && timestamp.compareTo(endTs) <= 0) {
    // 범위 내
}
```

### 6-2. 시간 그룹핑 (시간대별)
```java
String hourKey = timestamp.substring(0, 10); // yyyyMMddHH
```

### 6-3. 날짜 그룹핑 (일별)
```java
String dateKey = timestamp.substring(0, 8);  // yyyyMMdd
```

---

## 7. 통계/집계 패턴

### 7-1. 일치율 / 정확도
```java
int total = 0;
int correct = 0;

for (...) {
    total++;
    if (predicted == actual) correct++;
}

System.out.println(correct + "/" + total);
```

### 7-2. 평균값 (정수 처리)
```java
long sum = 0;
int count = 0;

for (...) {
    sum += value;
    count++;
}

long average = (count > 0) ? sum / count : 0;
```

### 7-3. 최댓값 / 최솟값
```java
int max = Integer.MIN_VALUE;
int min = Integer.MAX_VALUE;

for (int val : values) {
    max = Math.max(max, val);
    min = Math.min(min, val);
}
```

### 7-4. 빈도수 카운팅
```java
Map<String, Integer> countMap = new HashMap<>();
for (String item : list) {
    countMap.merge(item, 1, Integer::sum);
    // 또는
    countMap.put(item, countMap.getOrDefault(item, 0) + 1);
}
```

---

## 8. 정렬 패턴

### 8-1. List 정렬
```java
List<String> list = new ArrayList<>(map.keySet());
Collections.sort(list);  // 오름차순

// 내림차순
Collections.sort(list, Collections.reverseOrder());
```

### 8-2. 커스텀 정렬 (숫자 포함 문자열 등)
```java
list.sort((a, b) -> {
    int numA = Integer.parseInt(a.replaceAll("[^0-9]", ""));
    int numB = Integer.parseInt(b.replaceAll("[^0-9]", ""));
    return Integer.compare(numA, numB);
});
```

### 8-3. Map을 Value 기준 정렬
```java
List<Map.Entry<String, Integer>> entries = new ArrayList<>(map.entrySet());
entries.sort((a, b) -> b.getValue().compareTo(a.getValue())); // 내림차순
```

---

## 9. 파일 출력

### 9-1. 파일 쓰기
```java
BufferedWriter bw = new BufferedWriter(new FileWriter("OUTPUT.TXT"));
bw.write("결과: " + result);
bw.newLine();
bw.close();
```

### 9-2. PrintWriter (더 편리)
```java
PrintWriter pw = new PrintWriter(new FileWriter("OUTPUT.TXT"));
pw.println("line1");
pw.printf("%s/%d%n", name, count);
pw.close();
```

---

## 10. 전체 뼈대 코드 (복붙용)

### 🔹 콘솔형 (SUB1/SUB2 패턴)
```java
package com.lgcns.test;

import java.io.*;
import java.util.*;

public class RunManager {
    public static void main(String[] args) throws Exception {
        // 1. 파일 읽기
        BufferedReader br = new BufferedReader(new FileReader("DATA.TXT"));
        String line;
        Map<String, String> dataMap = new HashMap<>();
        while ((line = br.readLine()) != null) {
            line = line.trim();
            if (line.isEmpty()) continue;
            String[] parts = line.split("#");
            // 파싱 로직
        }
        br.close();

        // 2. (선택) 콘솔 입력
        BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));
        String input = stdin.readLine().trim();

        // 3. 처리 로직
        int total = 0, correct = 0;
        // ...

        // 4. 콘솔 출력
        System.out.println(correct + "/" + total);
    }
}
```

### 🔹 HTTP 서버형 (SUB3/SUB4 패턴)
```java
package com.lgcns.test;

import java.io.*;
import java.util.*;
import com.google.gson.*;
import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.handler.AbstractHandler;
import javax.servlet.http.*;

public class RunManager {
    private static Map<String, JsonObject> dataStore = new HashMap<>();
    private static Map<String, List<String>> configMap = new HashMap<>();

    public static void main(String[] args) throws Exception {
        // 1. 설정 파일 로드
        loadConfig();

        // 2. HTTP 서버 시작
        Server server = new Server(8080);
        server.setHandler(new AbstractHandler() {
            @Override
            public void handle(String target, Request baseRequest,
                    HttpServletRequest request, HttpServletResponse response)
                    throws IOException {
                switch (target) {
                    case "/data":
                        handleData(request, response);
                        break;
                    case "/query":
                        handleQuery(request, response);
                        break;
                }
                baseRequest.setHandled(true);
            }
        });
        server.start();
        server.join();
    }

    private static void loadConfig() throws Exception {
        BufferedReader br = new BufferedReader(new FileReader("CONFIG.JSON"));
        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = br.readLine()) != null) sb.append(line);
        br.close();

        JsonObject json = JsonParser.parseString(sb.toString()).getAsJsonObject();
        for (Map.Entry<String, JsonElement> entry : json.entrySet()) {
            List<String> list = new ArrayList<>();
            for (JsonElement e : entry.getValue().getAsJsonArray()) {
                list.add(e.getAsString());
            }
            configMap.put(entry.getKey(), list);
        }
    }

    private static String readBody(HttpServletRequest request) throws IOException {
        BufferedReader br = request.getReader();
        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = br.readLine()) != null) sb.append(line);
        return sb.toString();
    }

    private static void handleData(HttpServletRequest request,
            HttpServletResponse response) throws IOException {
        JsonObject data = JsonParser.parseString(readBody(request)).getAsJsonObject();
        String id = data.get("id").getAsString();
        dataStore.put(id, data);
        response.setStatus(200);
    }

    private static void handleQuery(HttpServletRequest request,
            HttpServletResponse response) throws IOException {
        JsonObject req = JsonParser.parseString(readBody(request)).getAsJsonObject();

        // 처리 로직
        JsonObject result = new JsonObject();
        result.addProperty("count", 0);

        response.setStatus(200);
        response.setContentType("application/json");
        response.getWriter().write(result.toString());
    }
}
```

---

## 💡 실전 팁

1. **상대경로 필수** — `new FileReader("DATA.TXT")` (절대경로 ❌)
2. **불필요한 출력 금지** — 디버그 `println` 제출 전 반드시 삭제
3. **프로그램 종료 조건** — SUB1/2는 자동종료, SUB3/4는 종료 없이 대기(`server.join()`)
4. **대소문자 정확히** — JSON key, endpoint path 모두 시험지와 **정확히** 일치
5. **dataType trim 필수** — JSON에 `"P "` 처럼 공백이 들어올 수 있음 → `.trim()`
6. **정수 처리** — 평균값 등은 `long / int` 정수 나눗셈 (문제에서 정수 보장)
7. **선행문항 복사** — SUB1→SUB2→SUB3→SUB4 순서로 소스 복사 후 확장


```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.handler.AbstractHandler;

import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

public class RunManager {

	// AI 에이전트한테 보고받은 데이터 저장
	static Map<String, MonitoringData> dataMap = new HashMap<>();
	
	// MODEL.JSON 정보를 저장
	static Map<String, List<String>> modelMap = new HashMap<>();
	
	
	public static void main(String[] args) throws Exception {
		
		// 서버 시작 전에 MODEL.JSON 파일 읽음
		loadModels();
		
		// Jetty HTTP 서버 생성
		// POST http://127.0.0.1:8080/performance
		// POST http://127.0.0.1:8080/monitoring
		Server server = new Server(8080);
		server.setHandler(new AbstractHandler() {

			@Override
			public void handle(String target, Request baseRequest, HttpServletRequest request,
					HttpServletResponse response) throws IOException, ServletException {
				if("/monitoring".equals(target) 
						&& "POST".equalsIgnoreCase(request.getMethod()) ) {
					
					handleMonitoring(request, response);
					baseRequest.setHandled(true);
					
				} else if("/performance".equals(target) 
						&& "POST".equalsIgnoreCase(request.getMethod())) {
					
					handlePerformance(request, response);
					baseRequest.setHandled(true);
					
				}
			}
		});
		
		server.start();
		server.join();
	}

	private static void loadModels() throws Exception {

	    BufferedReader br = new BufferedReader(new FileReader("MODELS.JSON"));
	    StringBuilder sb = new StringBuilder();
	    String line;

	    while ((line = br.readLine()) != null) {
	        sb.append(line);
	    }
	    br.close();

	    // JSON 파싱
	    JsonObject json = JsonParser.parseString(sb.toString()).getAsJsonObject();

	    for (Map.Entry<String, JsonElement> entry : json.entrySet()) {

	        List<String> list = new ArrayList<>();

	        for (JsonElement e : entry.getValue().getAsJsonArray()) {
	            list.add(e.getAsString());
	        }

	        modelMap.put(entry.getKey(), list);
	    }
	}
	
	
    static void handleMonitoring(HttpServletRequest request, HttpServletResponse response) throws IOException {

		String body = readBody(request);
		
		// Gson 파싱
		JsonObject json = JsonParser.parseString(body).getAsJsonObject();
		
        String agentId = json.get("agentId").getAsString();
        String requestId = json.get("requestId").getAsString();
        String timestamp = json.get("timestamp").getAsString();
        String dataType = json.get("dataType").getAsString();
        String dataValue = json.get("dataValue").getAsString();

        int latency = 0;

        // 4번: latency는 P 데이터일 때만 제공됨
        if ("P".equals(dataType) && json.has("latency")) {
            latency = json.get("latency").getAsInt();
        }

        addMonitoringData(agentId, requestId, timestamp, dataType, dataValue, latency);
		
		response.setStatus(HttpServletResponse.SC_OK);
	}
   
    static void handlePerformance(HttpServletRequest request,
            HttpServletResponse response) throws IOException {

		String body = readBody(request);
		
		// Gson 파싱
		JsonObject json = JsonParser.parseString(body).getAsJsonObject();
		
		String modelName = json.get("modelName").getAsString();
		String timeWindow = json.get("timeWindow").getAsString();
		
		int[] result = calculateAccuracy(modelName, timeWindow);
        // result[0] = correct
        // result[1] = total
        // result[2] = average latency
		
	    JsonObject res = new JsonObject();
	    res.addProperty("correct", result[0]);
	    res.addProperty("total", result[1]);
        res.addProperty("latency", result[2]);

	    response.setStatus(HttpServletResponse.SC_OK);
	    response.setContentType("application/json;charset=UTF-8");
	    response.getWriter().write(res.toString());
    }
    
    private static String readBody(HttpServletRequest request) throws IOException {
        BufferedReader br = request.getReader();
        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = br.readLine()) != null) sb.append(line);
        return sb.toString();
    }

	// 데이터 적재
	private static void addMonitoringData(String agentId,
            String requestId,
            String timestamp,
            String type,
            String value,
            int latency) {
		
		// 여러개의 AI agent가 AI 모델 사용할 수 있음
		// requestId는 agent마다 중복 가능 → agentId까지 포함해서 key 만들어야 안전
        String key = agentId + "#" + requestId;

        MonitoringData data = dataMap.get(key);

        if (data == null) {
            data = new MonitoringData();
            data.agentId = agentId;
            data.requestId = requestId;
        }

        if ("P".equals(type)) {
            data.predictValue = value;
            data.predictTimestamp = timestamp;
            data.latency = latency;
        } else if ("A".equals(type)) {
            data.actualValue = value;
        }

        dataMap.put(key, data);
	}
	
    static int[] calculateAccuracy(String modelName, String timeWindow) {

        int correct = 0;
        int total = 0;
        int latencySum = 0;

        List<String> agents = modelMap.get(modelName);

        if (agents == null) {
            return new int[]{0, 0, 0};
        }

        String start = timeWindow + "0000";
        String end = timeWindow + "5959";

        for (MonitoringData data : dataMap.values()) {

            if (!agents.contains(data.agentId)) continue;
            if (data.predictTimestamp == null) continue;

            if (data.predictTimestamp.compareTo(start) < 0 ||
                data.predictTimestamp.compareTo(end) > 0) {
                continue;
            }

            if (data.predictValue != null && data.actualValue != null) {
                total++;

                if (data.predictValue.equals(data.actualValue)) {
                    correct++;
                }
                
                // 4번: 대상 요청들의 latency 평균 산출
                latencySum += data.latency;
            }
        }
        
        int avgLatency = 0;

        if (total > 0) {
            avgLatency = latencySum / total;
        }

        return new int[]{correct, total, avgLatency};
    }
	
    static class MonitoringData {
    	String agentId;
        String requestId;
        String predictValue;
        String actualValue;
        String predictTimestamp;
        int latency;
    }
}
```
