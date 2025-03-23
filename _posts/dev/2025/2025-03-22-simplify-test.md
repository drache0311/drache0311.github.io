---
title: "Simplify test"
search: false
tags: [test, react]
last_modified_at: 2025-03-22T18:10:00
---
React 를 테스트하면서 생긴 고민이 있나요? mock api, act errors, test scope, better way to test...  
저는 아직도 많은 고민들을 간직하고 있는데요. 모든 고민들의 끝은 결국 테스트 코드를 어떻게 간단하게 할 수 있을까? 로 모입니다.    
복잡한 테스트 코드로 일어날 수 있는 모든 케이스들을 커버하는게 좋을까요? jest 로 E2E 를 하고 계신가요?  
물론 버그들을 방지하는데 도움이 되겠지만, 얻는 이점보다는 잃는 이점이 많아 보입니다.    
테스트가 리팩토링을 방해하고 있진 않나요? 함께 일하는 동료가 코드를 이해하기 쉬운가요? 테스트가 점점 느려지는건요?  
이런 고민들은 수 많은 개발자들이 겪고 있으며, 수 많은 글이 존재하고 있습니다. 이 말은 정답은 없지만 좋은 방향성을 찾아가기에 꾸준히 글들이 나오지 않을까
싶습니다.  
저도 아직 많은 고민이 있고 해결방법을 찾아가고 있지만, 이 중 찾아낸 좋은 방향성을 가진 방법들이 테스트 코드를 작성하는데 많은 도움을 주어 소개하고자 합니다.
이 주제는 Simplify Test 에 대한 시리즈 글이며, 1번(Avoid nesting test) 에 대해 다루겠습니다.

1. Avoid nesting test
2. [Do not mock api directly]()
3. [What is act errors?]()

# Avoid nesting test
중복된 코드를 어떻게 관리하고 계신가요? 실제 구현체 코드에서는 중복된 코드를 줄여가는 습관이 있으실 겁니다. 이는 코드를 더 이해하기 쉽게하는데 도움을 줍니다.  
하지만 테스트 코드에서는요? 테스트 코드들에서도 점점 중복되는 코드가 느껴지실 겁니다.  
이런 상황에서 중복된 코드들을 `beforeEach`에서 사용했나요? 또는 `describe` 로 나누기도 했나요?  
자연스럽게, 테스트 코드에서도 중복된 코드들을 처리하고자 했을 것입니다. 과연 구현체 코드처럼 도움이 될까요?  
이 주제는 Kent C. Dodds의 [Avoid Nesting when you're Testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing)
글에서 자세히 다루고 있으며 저의 생각도 나누어 보고자 합니다.

위 글을 참조하여 테스트를 위한 리액트 컴포넌트를 가져와봤습니다.

```tsx
const Login = ({onSubmit}:{onSubmit:{name:string, password:string}}) => {
    const [error, setError] = useState('')
    const [userName, setUserName] = useState('')
    const [password, setPassword] = useState('')

    const handleSubmit = () => {
        if (!username) {
            setError('username is required')
        } else if (!password) {
            setError('password is required')
        } else {
            setError('')
            onSubmit({ username, password })
        }
    }

    return (
        <div>
            <div>
                <label htmlFor="usernameInput">Username</label>
                <input id="usernameInput" value={userName} onChange={setUserName}/>
            </div>
            <div>
                <label htmlFor="passwordInput">Password</label>
                <input id="passwordInput" type="password" value={password} onChange={setPassword}/>
            </div>
            <button onClick={handleSubmit}>Submit</button>
            {error ? <div role="alert">{error}</div> : null}
        </div>
    )
}
```

간단합니다. 이름과 비밀번호를 입력하여 로그인을 하는 컴포넌트입니다.  
여러분은 이 컴포넌트를 어떻게 테스트 하실건가요?

```tsx
// login.ts
const USER = { name : 'yongbeom' , password : 'dragontiger' }
describe('Login Tests', () => {
    let handleSubmit, clickSubmit
    
    beforeEach(() => {
        handleSubmit = jest.fn()
        render(<Login onSubmit={handleSubmit}/>)
        clickSubmit = userEvent.click(screen.getByText(/submit/i))
    })

    it('should type user name', async () => {
        await userEvent.type(screen.getByLabelText(/username/i), USER.name)
        
        expect(screen.getByLabelText(/username/i)).toHaveValue(USER.name)
    })

    it('should type password', async () => {
        await userEvent.type(screen.getByLabelText(/password/i), USER.password)
        
        expect(screen.getByLabelText(/password/i)).toHaveValue(USER.password)
    })
    
    describe('when username and password is valid', () => {
        beforeEach(async () => {
            await userEvent.type(screen.getByLabelText(/username/i), USER.name)
            await userEvent.type(screen.getByLabelText(/password/i), USER.password)
            await clickSubmit()
        })

        it('should call handleSubmit', async () => {
            expect(handleSubmit).toHaveBeenCalledTimes(1)
            expect(handleSubmit).toHaveBeenCalledWith(USER)
        })
    })

    describe('when username is not valid', () => {
        beforeEach(async () => {
            await userEvent.click(screen.getByText(/submit/i))
        })

        it('should not call handleSubmit', async () => {
            expect(handleSubmit).not.toHaveBeenCalled()
        })

        it('should show an error message', () => {
            expect(errorMessage).toHaveTextContent(/username is required/i)
        })
    })

    describe('when password is not valid', () => {
        beforeEach(async () => {
            await userEvent.type(screen.getByLabelText(/username/i), USER.name)
            await userEvent.click(screen.getByText(/submit/i))
        })

        it('should not call handleSubmit', async () => {
            expect(handleSubmit).not.toHaveBeenCalled()
        })

        it('should show an error message', () => {
            expect(errorMessage).toHaveTextContent(/password is required/i)
        })
    })
})
```

읽기 쉬우셨나요? 그럴수도 있고, 아닐 수도 있습니다. 아니라면 어떤 점이 읽기 어렵게 했을까요?    
여러분도 이렇게 테스트 상황을 describe 로 나누거나 공통된 작업을 beforeEach 에서 하셨나요? 변수들을 선언 후 beforeEach 에서 재할당을 하셨나요?  
이 테스트 코드는 Login 컴포넌트에 대한 100% 신뢰를 주고 있습니다. 테스트 상황별로 describe 를 나누어 특정 상황에 대한 코드를 찾을 수 있거나 원하는 기능
만을 위한 테스트를 찾을 수도 있습니다. 하지만 Kent C 의 말처럼 이러한 테스트 방식을 저도 선호하지 않는데요.   
지금 개발을 진행중인 Frontiers 에서도 `describe` 를 통한 테스트 그룹핑, `beforeEach` 를 통한 테스트 셋업을 한 후 `it` 에서 테스트를 하고 있습니다.
위와 같은 예시처럼 작성한 테스트 코드가 있었는데, 개발에 대한 피로도와 복잡함을 많이 상승시켜 테스트에 대한 어려움을 주었습니다.
중첩된 테스트를 많이 지양하고 있음에도, 최소한의 중첩이 존재하고 있고, 그에 의해 jest 를 효율적으로 사용하고 있는가? 또는 이 글에서 다루는 주제를 관철하고 있는가?
에 대한 점은 저에게는 아직 아쉬운 과제로 남아있습니다. 팀원들간의 테스트 목적의 방향성, 개발 역량, 사고 방식, 클린 코드에 대한 관심 등간의 차이로 인해 좁혀나가고 싶지만 쉽지는 않습니다.
저 또한 첫 팀, 심지어는 비 IT 회사의 환경이기에 부족한 경험과 개발 관심도가 다른 팀원들과의 적당한 타협점을 찾는 것이 가장 어려운 것 같습니다.    
하지만 꾸준히 좋은 방향을 찾아가고, 공유를 하다보면 언젠가 좋은 환경, 팀은 내가 만들어갈 수 있지 않을까 하는 기대가 있습니다. 
그럼 이제 위와 같은 코드가 어떤 복잡함을 주고, 그 이유와 해소 방법에 대해 위 글을 참조하여 이야기 하겠습니다.

# Nesting
그렇다면 어떤게 중첩된 코드일까요? 위 코드는 알다시피 jest 로 작성되었지만, 다른 테스트 프레임워크에서도 비슷할 것입니다.  
앞서 말했던 그룹핑을 위한 `describe`, 테스트 셋업을 위한 `beforeEach` 의 사용을 말하는 것인데요.  
저는 이것들의 사용이 최소한이 되어야 한다고 생각합니다.  
그렇다면 중첩된 코드가 테스트를 어떻게 복잡하게 할까요? 작은 범위의 코드에서부터 찾아볼 수 있습니다.  

```tsx
it('should call handleSubmit', async () => {
    expect(handleSubmit).toHaveBeenCalledTimes(1)
    expect(handleSubmit).toHaveBeenCalledWith(USER)
})
```

이 코드가 이해하기 쉬워보이나요? `handleSubmit` 을 호출한다는 것은 명확해 보입니다.  
하지만 handleSubmit 이 어디에서부터 왔고, 어떤 사전작업을 거치고 왔을까요? 이것만 보면은 알 수가 없습니다.
테스트 코드를 작성하는 당시의 당신은 알고 있을 것입니다. 하지만 다른 사람들은요? 또는 시간이 지난 후 봤을 때도 기억하고 있을까요?  

```tsx
let handleSubmit

beforeEach(() => {
    handleSubmit = jest.fn()
    render(<Login onSubmit={handleSubmit}/>)
    clickSubmit = userEvent.click(screen.getByText(/submit/i))
})
```

handleSubmit은 테스트 파일의 가장 위에서 선언되어 있으며 beforeEach 에서 재할당되어 테스트 할 컴포넌트에 props 로 전달되고 있습니다.  
우리가 보고 있던 테스트 코드가 1000줄 중 900 줄에 있었다면, handleSubmit 을 찾기 위해 900 줄 앞으로 가야합니다.  

```tsx
beforeEach(async () => {
    await userEvent.type(screen.getByLabelText(/username/i), USER.name)
    await userEvent.type(screen.getByLabelText(/password/i), USER.password)
    await clickSubmit()
})
```

그렇다면, 사전 작업은요? 해당 테스트 코드를 그룹핑하고 있는 describe 밑의 beforeEach 에 정의되어 있습니다.  
clickSubmit 을 놓쳤었다면, 다시 찾으러 가봐야겠군요.  
눈치채셨나요? 저 테스트 코드를 이해하기 위해서는, 우리는 얼마나 많을지 모르는 line 을 지나며 찾아야하고, 다시 테스트 코드로 돌아와야 합니다.  
이 점은 당신의 눈을 피로하게 만들며, 이 다음의 테스트 코드를 작성할 때에도 항상 생각하게 만들 것입니다.
이처럼 `beforeEach` 나 `describe` 로 중첩되어진 코드를 이어받은 `it` 에서는 오히려 당신을 힘들게 합니다.

# Abstract
Kent C 는 `handleSubmit` 이나 `clickSubmit` 같은 유틸은 좋다고 생각하지만, 이런 작은 테스트에서는 오히려 유지보수에 불편함을 준다고 합니다.  
더불어 변수 재할당은 `anti-pattern` 이라 생각하지만 이미 시간이 지난 프로젝트에서 추가적인 린팅 규칙은 최선의 해결책은 아닐 수 있다고 합니다.  
Frontiers 를 하면서도 느끼지만, 처음 정한 린팅 규칙도 완전히 이루어지지 않거나 오히려 변하기도 하는 프로젝트에서는 부담스럽다고 생각합니다.  
그렇다면 어떻게 하는게 좋을까요?

# Inline it
중첩한 코드들을 it 테스트 안에서 직접 사용하는 것입니다.  
밑의 예시 코드를 함께 보겠습니다.

```tsx
// login.ts
test('calls onSubmit with the username and password when submit is clicked', () => {
	const handleSubmit = jest.fn()
	const { getByLabelText, getByText } = render(
		<Login onSubmit={handleSubmit} />,
	)
	const user = { username: 'yongbeom', password: 'dragontiger' }

	userEvent.type(getByLabelText(/username/i), user.username)
	userEvent.type(getByLabelText(/password/i), user.password)
	userEvent.click(getByText(/submit/i))

	expect(handleSubmit).toHaveBeenCalledTimes(1)
	expect(handleSubmit).toHaveBeenCalledWith(user)
})
```

이 예를 보면 앞서 중첩되었던 코드들이 한 테스트에 들어가 있습니다. `describe` 도 없어졌고, `it` 가 아닌 `test` 로 변경된 점도 있습니다.
Kent C 는 이 테스트 파일이 Login 컴포넌트를 테스트 하는 파일이며, 모든 테스트들이 독립적인 테스트가 되었다면, describe 는 더이상 필요 없으며 
it 보다는 test 를 더 선호한다고 합니다.  
it 와 test는 취향의 영역이겠지만, 한글이 모국어인 저로서는 별 차이가 없어보입니다.  
어쨋거나 이 같은 방식은 각 테스트에서 무슨 일이 일어나는지 이해하는 능력을 크게 향상시켜, 불필요하게 스크롤할 필요가 없도록 합니다.  
만약 이 컴포넌트에 수십 개의 추가 테스트가 있었다면, 그 이점은 더욱 강력해졌을 것입니다.


# AHA(Avoid Hasty Abstractions)
언제부터인가 자연스럽게 [Dry](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 한 코드를 무의식적으로 지향하고 있던 것 같습니다.  
그래서 테스트 코드에서도 반복되는 코드를 beforeEach 나 describe 로 감싸지 않으면 마음이 불편하고 그로인해 누가 가르쳐주지 않아도 자연스럽게 사용했던 것 같습니다.  
이제 더 이상 그럴 필요가 없습니다. `AHA`는 이렇게 말합니다.  
>prefer duplication over the wrong abstraction and optimize for change first.  

우리의 Login 컴포넌트의 테스트는 그대로 두어도 되지만, 만약 더 복잡해지고 코드의 중복이 늘어난다고 가정해봅시다.  
그럼 어떻게 해야할까요? beforeEach 를 사용해야할까요? 그것이 beforeEach 의 기능 아닌가요?  
하지만 우리는 위에서 언급한 것처럼 변수 재할당 같은 것을 조심해야합니다.  
그럼 우리의 테스트 코드에서 중복된 코드를 어떻게 공유할 수 있을까요? AHA! 함수를 사용하면 되겠네요!

```tsx
// 이보다 나은 예시가 생각나지 않아 참고한 글의 예시를 가져왔습니다.
// here we have a bunch of setup functions that compose together for our test cases
// I only recommend doing this when you have a lot of tests that do the same thing.
// I'm including it here only as an example. These tests don't necessitate this
// much abstraction. Read more: https://kcd.im/aha-testing
const setup = () => {
	const handleSubmit = jest.fn()
	const utils = render(<Login onSubmit={handleSubmit} />)
	const user = { username: 'yongbeom', password: 'dragontiger' }
	const changeUsernameInput = (value) => userEvent.type(utils.getByLabelText(/username/i), value)
	const changePasswordInput = (value) => userEvent.type(utils.getByLabelText(/password/i), value)
	const clickSubmit = () => userEvent.click(utils.getByText(/submit/i))
	return {
		...utils,
		handleSubmit,
		user,
		changeUsernameInput,
		changePasswordInput,
		clickSubmit,
	}
}

const setupSuccessCase = () => {
	const utils = setup()
	utils.changeUsernameInput(utils.user.username)
	utils.changePasswordInput(utils.user.password)
	utils.clickSubmit()
	return utils
}

const setupWithNoPassword = () => {
	const utils = setup()
	utils.changeUsernameInput(utils.user.username)
	utils.clickSubmit()
	const errorMessage = utils.getByRole('alert')
	return { ...utils, errorMessage }
}

const setupWithNoUsername = () => {
	const utils = setup()
	utils.changePasswordInput(utils.user.password)
	utils.clickSubmit()
	const errorMessage = utils.getByRole('alert')
	return { ...utils, errorMessage }
}

test('calls onSubmit with the username and password', () => {
	const { handleSubmit, user } = setupSuccessCase()
	expect(handleSubmit).toHaveBeenCalledTimes(1)
	expect(handleSubmit).toHaveBeenCalledWith(user)
})

test('shows an error message when submit is clicked and no username is provided', () => {
	const { handleSubmit, errorMessage } = setupWithNoUsername()
	expect(errorMessage).toHaveTextContent(/username is required/i)
	expect(handleSubmit).not.toHaveBeenCalled()
})

test('shows an error message when password is not provided', () => {
	const { handleSubmit, errorMessage } = setupWithNoPassword()
	expect(errorMessage).toHaveTextContent(/password is required/i)
	expect(handleSubmit).not.toHaveBeenCalled()
})
```

이제 우리는 이 간단한 함수를 사용하는 수십 개의 테스트를 가질 수 있으며, 이러한 함수들을 조합하면 이전에 사용했던 중첩된 beforeEach와 유사한 동작을 구현할 수 있다는 점도 주목할 만합니다.
간단한 테스트에서는 이런 과정이 필요하지 않지만, 중요하다고 생각하는 테스트에서는 이러한 테스트 방식을 적용하고 테스트의 효율성을 높여보는 연습을 해보아야겠습니다.  
[AHA](https://kentcdodds.com/blog/aha-testing) 에 관해서는 이 글을 추천드립니다.

# Grouping
...

# CleanUp
...

