# Case Study 4: Social Media Feed with Infinite Scrolling
### Abhay Nanduri
### AIE B
### CH.EN.U4AIE21130


First Install Dependencies
```bash
npm install
```

Then start the app
```bash
npm start
```

**I am using r/tehnology subreddit to fetch article requests.**

**If you are about to use NewsAPI then make a `.env` with NewsFeed API Key and copy `newsapicode.js` to `NewsFeed.js`.**


## Implementation of Infinite Scrolling in React

1. How would you implement infinite scrolling in a React component?

Infinite scrolling can be implemented using the `useEffect` and `useState` hooks in React, along with the Intersection Observer API. Here's a basic implementation:

```javascript
import React, { useState, useEffect, useRef } from 'react';

const InfiniteScrollComponent = () => {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const loaderRef = useRef(null);

  useEffect(() => {
    const options = {
      root: null,
      rootMargin: '20px',
      threshold: 1.0
    };

    const observer = new IntersectionObserver(handleObserver, options);
    if (loaderRef.current) {
      observer.observe(loaderRef.current);
    }

    return () => {
      if (loaderRef.current) {
        observer.unobserve(loaderRef.current);
      }
    };
  }, []);

  const handleObserver = (entities) => {
    const target = entities[0];
    if (target.isIntersecting) {
      setPage((prev) => prev + 1);
    }
  };

  useEffect(() => {
    fetchMoreItems();
  }, [page]);

  const fetchMoreItems = async () => {
    // Fetch logic here
  };

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>{item.title}</div>
      ))}
      <div ref={loaderRef}>Loading...</div>
    </div>
  );
};
```

2. Describe how to fetch and display additional posts as the user scrolls.

To fetch and display additional posts, we can use the `useEffect` hook to trigger a fetch when the page number changes:

``` javascript
useEffect(() => {
  fetchMoreItems();
}, [page]);

const fetchMoreItems = async () => {
  setLoading(true);
  try {
    const response = await fetch(`https://api.reddit.com/r/technology/hot.json?limit=10&after=t3_${items[items.length - 1]?.id || ''}`);
    const data = await response.json();
    const newItems = data.data.children.map(child => child.data);
    setItems(prevItems => [...prevItems, ...newItems]);
  } catch (error) {
    console.error('Error fetching posts:', error);
  } finally {
    setLoading(false);
  }
};
```

3. How can you optimize the loading of posts to improve performance and user experience?

To optimize loading and improve performance:

- Implement virtualization for large lists
- Use memoization to prevent unnecessary re-renders
- Implement debouncing for scroll events

Example of virtualization using react-window:

``` javascript
import { FixedSizeList as List } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>
    {items[index].title}
  </div>
);

const VirtualizedList = () => (
  <List
    height={400}
    itemCount={items.length}
    itemSize={35}
    width={300}
  >
    {Row}
  </List>
);
```

4. Explain how you would handle loading states and display a spinner while new posts are being fetched.

To handle loading states and display a spinner:

``` javascript
import { ClipLoader } from 'react-spinners';

// ... inside the component
return (
  <div>
    {items.map((item) => (
      <div key={item.id}>{item.title}</div>
    ))}
    {loading && (
      <div className="spinner-container">
        <ClipLoader color="#000000" loading={loading} size={50} />
      </div>
    )}
    <div ref={loaderRef}></div>
  </div>
);
```

5. What are the potential challenges with infinite scrolling, and how would you address them?

Challenges and solutions:

a. Performance issues with large datasets:
   - Implement virtualization (as shown above)
   - Use pagination or "load more" buttons for very large datasets

b. SEO problems due to lack of pagination:
   - Implement hybrid approach with both infinite scroll and traditional pagination

c. Difficulty in reaching footer content:
   - Add a "Back to Top" button

``` javascript
const BackToTopButton = () => {
  const [visible, setVisible] = useState(false);

  const toggleVisible = () => {
    const scrolled = document.documentElement.scrollTop;
    if (scrolled > 300) {
      setVisible(true);
    } else if (scrolled <= 300) {
      setVisible(false);
    }
  };

  const scrollToTop = () => {
    window.scrollTo({
      top: 0,
      behavior: 'smooth'
    });
  };

  useEffect(() => {
    window.addEventListener('scroll', toggleVisible);
    return () => {
      window.removeEventListener('scroll', toggleVisible);
    };
  }, []);

  return (
    <button
      onClick={scrollToTop}
      style={{display: visible ? 'inline' : 'none'}}
    >
      Back to Top
    </button>
  );
};
```

d. Browser history management:
   - Use the History API to update the URL as the user scrolls

``` javascript
useEffect(() => {
  window.history.pushState(null, '', `?page=${page}`);
}, [page]);

```
By addressing these challenges, you can create a more robust and user-friendly infinite scrolling implementation.
