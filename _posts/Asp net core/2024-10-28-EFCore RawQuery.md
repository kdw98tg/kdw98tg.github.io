---
title: "[ASP.NET Core] ASP.Net Core 에서 RawQuery로 데이터 조회하기"

categories:
  -  ASP .NET Core
  
tags:
  - [Web, ASP.NET Core, C#, RawQuery]

toc: true
toc_sticky: true

published: true

date: 2024-10-28
last_modified_at: 2024-10-28
---

## EntityFrameWorkCore 의 ORM

Asp.net core 는 `EntityFrameworkCore`라는 ORM 라이브러리를 제공합니다. ORM 이라고 하면, 데이터베이스의 엔티티들을 객체지향적으로 바꾸어 쿼리를 짜지 않고, 지원하는 언어만 가지고 데이터를 조회, 수정, 삭제 할 수 있는 프레임워크를 말합니다.

C#에서는 보통 Linq 식을 사용하여 데이터를 조회하게 됩니다. 형식은 다음과 같습니다.

```csharp
var user = await context.User.FindAsync(userId);
```

간단한 조건의 데이터를 찾을때는 정말 간편하지만, 다중 조인이 들어가거나, 조건식이 복잡할 경우에는 쿼리를 직접 입력하여 데이터를 가져오는것이 더 편할때가 있습니다. 또한, C#의 Linq식에는 Left Join 등의 기능을 직접적으로 지원하고 있지 않아서 해당 기능을 구현하려면 굉장히 복잡한 식을 사용해야 합니다.

이번 포스팅에서는 `EFCore` 에서 RawQuery를 가지고 데이터를 가져오는 방식에 대해 이야기 합니다.

## RawQuery로 데이터 조회하기

RawQuery를 사용하기 위한 라이브러리는 `MySqlConnector`를 사용합니다. `MySql.dll` 과 비슷하게, Command 객체에 쿼리를 입력하고, Reader 객체가 가져온 데이터를 읽는 형식 입니다.

### Query 작성하기

우선, Query를 작성 합니다. 간단하게 Id 를 기준으로 User 를 찾는 쿼리를 작성합니다. (이정도의 조회라면 ORM 을 사용하는것이 편하나, 해당 포스팅에서는 예를 들기위해 사용했습니다.)

```csharp
[HttpGet]
public async Task<ActionResult<UserDTO>> GetUser(string _userId)
{
	if(_userId == null)
	{
		return NotFound();
	}

	var sqlQuery = @"
	SELECT * FROM User Where User_Id = @userId
	"
}
```

위와같이 Query를 작성합니다. userId 는 Parameter 객체를 사용하여 바인딩 해줍니다.

### Command 객체에 담아서 보내기

다음은 해당 쿼리를 Command 객체에 담아서 Mysql 서버에 보내줍니다. 또한, `MySqlParameter`객체를 사용하여 Query 문자열에 값을 바인딩 해줍니다.

```csharp
[HttpGet]
public async Task<ActionResult<UserDTO>> GetUser(string _userId)
{
	if(_userId == null)
	{
		return NotFound();
	}

	var sqlQuery = @"
	SELECT * FROM User Where User_Id = @userId
	"

	using(var command = context.Database.GetDbConnection().CreateCommand())
	{
		command.CommandText = sqlQuery;
		command.Parameters.Add(new MySqlParameter("userId", _userId))

		context.Database.OpenConnection();//커넥션 열기
	}
}
```

위와같이 쿼리에 값을 바인딩 하고, `OpenConnection()`을 사용하여 데이터베이스와 연결합니다.

### Reader로 값 읽어서 모델클래스에 바인딩

이제, 값을 읽어야 합니다. `Reader`객체를 통하여 값을 읽어줍니다. 여기서 주의할 점은 데이터베이스의 컬럼명을 그대로 입력해야 한다는 것입니다. (Alias를 통해 별칭을 사용했다면 해당 별칭을 사용)

```csharp
[HttpGet]
public async Task<ActionResult<UserDTO>> GetUser(string _userId)
{
	if(_userId == null)
	{
		return NotFound();
	}

	var sqlQuery = @"
	SELECT * FROM User Where User_Id = @userId
	"

	UserDTO userDTO = new UserDTO();//값을 담아 보낼 DTO 생성

	using(var command = context.Database.GetDbConnection().CreateCommand())
	{
		command.CommandText = sqlQuery;
		command.Parameters.Add(new MySqlParameter("userId", _userId))

		context.Database.OpenConnection();//커넥션 열기

		using(var reader = await command.ExecuteReaderAsync())
		{
			while(await reader.ReadAsync())
			{
				 var userData = new User()
				 {
					 User_Id = reader["User_Id"],
					 User_Name = reader["User_Name"],
				 };
				 userDTO = ModelToDTO(userData);
			} 
		}
		
		context.Database.CloseConnection();
	}
	
	return Ok(userDTO);
}
```

위의 코드는 유저의 정보를 가지고 와서, DTO 클래스에 매핑한 후, 클라이언트에 반환하도록 하는 코드 입니다.

그리고 반드시 `OpenConnection()`으로 커넥션을 열었다면, `CloseConnection()`으로 연결을 끊어줘야 합니다. (해당 문제를 방지하기 위해서 언어차원에서 using 이라는 키워드를 제공하지만, 혹시 모르니 명시적으로 코드를 넣어줍시다.)

