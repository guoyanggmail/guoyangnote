---
name: Remotion
description: Guidelines and best practices for creating Remotion video applications using React. Always use frame-based animations, pure components, and remotion's hooks.
---

# Remotion Video App Guidelines

This is a remotion based video app that uses React to render videos.
Full remotion docs can be found here: https://www.remotion.dev/docs/. Consult these docs often if you're uncertain.

## Project structure
The Root file is usually named "src/Root.tsx" and looks like this:

```tsx
import {Composition} from 'remotion'; 
import {MyComp} from './MyComp'; 

export const Root: React.FC = () => { 
  return ( 
    <> 
      <Composition id="MyComp" component={MyComp} durationInFrames={120} width={1920} height={1080} fps={30} defaultProps={{}} /> 
    </> 
  ); 
};
```

A `<Composition>` defines a video that can be rendered. It consists of a React "component", an "id", a "durationInFrames", a "width", a "height" and a frame rate "fps". The default frame rate should be 30. The default height should be 1080 and the default width should be 1920.

Inside a React "component", one can use the `useCurrentFrame()` hook to get the current frame number. Frame numbers start at 0.

```tsx
export const MyComp: React.FC = () => { 
  const frame = useCurrentFrame(); 
  return <div>Frame {frame}</div>; 
};
```

## Special Tags

If a video is included in the component it should use the `<OffthreadVideo>` tag.
```tsx
import {OffthreadVideo} from 'remotion'; 
export const MyComp: React.FC = () => { 
  return <div><OffthreadVideo src="https://remotion.dev/bbb.mp4" style={{width: '100%'}} /></div>; 
};
```

If an non-animated image is included In the component it should use the `<Img>` tag.
```tsx
import {Img} from 'remotion'; 
export const MyComp: React.FC = () => { 
  return <Img src="https://remotion.dev/logo.png" style={{width: '100%'}} />; 
};
```

If audio is included, the `<Audio>` tag should be used.
```tsx
import {Audio} from 'remotion'; 
export const MyComp: React.FC = () => { 
  return <Audio src="https://remotion.dev/audio.mp3" />; 
};
```

## Layering & Sequencing

If two elements should be rendered on top of each other, they should be layered using the `AbsoluteFill` component from "remotion".

Any Element can be wrapped in a `Sequence` component from "remotion" to place the element later in the video.
```tsx
import {Sequence} from 'remotion'; 
export const MyComp: React.FC = () => { 
  return ( 
    <Sequence from={10} durationInFrames={20}> 
      <div>This only appears after 10 frames</div> 
    </Sequence> 
  ); 
};
```

For displaying multiple elements after another, the `Series` component from "remotion" can be used.

For displaying multiple elements after another another and having a transition inbetween, the `TransitionSeries` component from "@remotion/transitions" can be used.

## Animation & Helpers

Remotion includes an `interpolate()` helper that can animate values over time.
```tsx
import {interpolate} from 'remotion'; 
export const MyComp: React.FC = () => { 
  const frame = useCurrentFrame(); 
  const value = interpolate(frame, [0, 100], [0, 1], { 
    extrapolateLeft: 'clamp', 
    extrapolateRight: 'clamp', 
  }); 
  return <div>Frame {frame}: {value}</div>; 
};
```

Remotion includes a `spring()` helper that can animate values over time.
```tsx
import {spring, useCurrentFrame, useVideoConfig} from 'remotion'; 
export const MyComp: React.FC = () => { 
  const frame = useCurrentFrame(); 
  const {fps} = useVideoConfig(); 
  const value = spring({ 
    fps, 
    frame, 
    config: { damping: 200, }, 
  }); 
  return <div>Frame {frame}: {value}</div>; 
};
```

## Remotion Components vs Interactive React Components
Remotion Components:
- Are rendered frame-by-frame to create videos
- Cannot have user interactions (no onClick, onHover, etc.)
- Cannot use hooks like useState for interactivity
- Must be deterministic - same input always produces same output
- Animations are driven by the current frame number

State Management
- Remotion: Use useCurrentFrame() to drive animations
- Normal React: Use useState() for interactive state

Animations
- Remotion: Use interpolate() or spring() based on frame number
- Normal React: Use CSS transitions, animation libraries, or requestAnimationFrame

Effects
- Remotion: Avoid useEffect - calculations should be pure based on frame
- Normal React: Use useEffect for side effects and subscriptions

### Best Practices for Remotion Components
1. Always use frame-based animations - Never rely on time-based effects
2. Keep components pure - No side effects or external data fetching
3. Use Remotion's hooks - useCurrentFrame(), useVideoConfig(), etc.
4. Leverage Sequences - For timing different elements
5. No interactive elements - Remove all event handlers from UI components
6. Deterministic rendering - Ensure consistent output for video rendering
