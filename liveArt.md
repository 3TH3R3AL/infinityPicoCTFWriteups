# Live Art - PicoCTF Writeup

## Description:

There's nothing quite as fun as drawing for an audience. So sign up for LiveArt today and show the world what you can do.

## Site Overview:

> Note: the site is made with react

##### /

Landing page with options to go to editor, settings, or fan-mail. Has option to join stream.

##### /editor

Open canvas with ability to draw on it

##### /settings

Allows you to set stream name and allow connections from viewers

##### /link-submission

Bot endpoint, can submit links here. 

##### /error

Error page

##### /drawing/:page

View drawing stream

## Solution

I only partially solved this challenge during the competition, but I was very close, so I decided to make a writeup anyway. There is a lot to unpack with the code, and I had many ideas that went nowhere, however, for purposes of this writeup, I am going to stick to only the relevant code. As I investigated the source code, I noticed that the error page took things from the URL query string, which I thought was suspicious. Additionally, as I played arround with the error endpoint, (its triggered based when the /drawing endpoint gets too small) I noticed that I would sometimes get an error when I changed the size of the page. The error endpoint uses the ```useHashParams``` custom hook, which stores the hash params in react's state. However, when the page is resized back to normal, the page goes back to the /drawing endpoint, while the state from the error page exists, allowing us to manipulate react's state. This is important because the state is used in the viewer component to create an element:

```javascript

export const Viewer = (props: Props) => {
    const [dimensions, updateDimensions] = React.useReducer(
        (canvasDimensions: Dimensions, windowDimensions: Dimensions) => {
            const newScale = Math.floor(Math.min(
                (windowDimensions.width / baseResolution.width),
                (windowDimensions.height / baseResolution.height))
            );

            const desiredDimensions = { width: baseResolution.width * newScale, height: baseResolution.height * newScale };

            if (desiredDimensions.width !== canvasDimensions.width || desiredDimensions.height !== canvasDimensions.height) {
                return desiredDimensions;
            } else {
                return canvasDimensions;
            }
        },
        baseResolution
    );

    React.useEffect(() => {
        const listener = () => {
            updateDimensions({ width: window.innerWidth - 100, height: window.innerHeight - 200 });
        }

        window.addEventListener("resize", listener);

        return () => {
            window.removeEventListener("resize", listener);
        }
    }, []);
    return (
        <div>
            <h1>Viewing</h1>
            
            <img src={props.image} { ...dimensions }/>
        </div>
    )
}

```

The important part of this code is the { ...dimensions }. This is the JS spread operator, and it means that if we manipulate the dimensions state, we can write arbitrary attributes to the img tag. If we could write, say "onLoad" or "onError", we could get xss and get the flag. However, react does not like this, and stops us from using these event listeners. This is as far as I got in the actual competion, I knew there was probably an attribute that would allow me to get xss, but I didn't know what it was, and was already really burnt out from non stop focus on the challenge. Also, by this point, my team had only two challenges to go, this and solfire, but in order to get top 3, we would need to solve both. Thus, not knowing where to go, and not even sure whether this was the real path to the exploit, I somewhat stopped working on the challenge, with the idea of giving myself a break to think about it. This had worked with noted, but this time it did not. It turns out that the attribute I was seeking was only two letters: "is". This attribute would make react treat it as a custom element, thus allowing me to pass onerror attributes and get xss. 

## Review

Overall, this was an amazing challenge with a very sophisticated exploit required. The one thing that bugs me about this challenge was that I had the idea of potentially bruteforcing it by simply testing many different attribute names until one worked. However, I didn't implement this because it seemed like a lot of effort and time for an unlikely payoff. However, it would have worked had I tried it. It just goes to show, when it comes to hacking, no idea is too crazy. 