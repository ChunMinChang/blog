# blog
Personal blog for development notes.
For convenience sake, I fork from
[hmfaysal/hmfaysal-omega-theme](https://github.com/hmfaysal/hmfaysal-omega-theme)
to log my working notes.

## Requirments
- Category
- Tags

## Run
- ```$ bundle exec jekyll serve```

## TO-DO
- Add table-of-content in post
- Remove the animation in the beginning of the page
- Survey: Move to hexo?
  - [themes here](https://hexo.io/themes/)
    - [Hiker](https://itimetraveler.github.io/hexo-theme-hiker/)
    - [Hipaper](https://itimetraveler.github.io/hexo-theme-hipaper/)
    - [Hiero](https://itimetraveler.github.io/hexo-theme-hiero/)
    - [Overdose](https://hyunseob.github.io/)
    - [Minos](http://blog.zhangruipeng.me/hexo-theme-minos/)
  - Integration with mathjax
    - http://cyruschiu.github.io/2016/07/02/github-pages-with-hexo/
    - http://liuhongjiang.github.io/hexotech/2015/08/12/mathjax-in-hexo/

### Posts
- Derive _rounding_ approach for _Fibonacci_ numbers
- Derive the time-complexity for different algorithms in _Fibonacci_ series
- Explain how _weak ptr_ works and its use case
  - Implement ```WeakPtr``` and make it work with my ```RefPtr```
  - Write sample code to explain why we need to use ```WeakPtr```
- Audio Seamless Looping
  - Need to rewrite the ffi analyser first
- Music Blinking LED
- Rust Notes
  - Common
    - Understand borrowing by read-write concepts
    - How to use phantom data
  - Common Error
    - Don't use the pointers of instances that will be returned from functions
  - FFI to C library
    - A Rust Library based on native C APIs
    - Using single-element (tuple) struct to wrap native types
    - How to wrap a callback from external library
  - Taste
    - Error passing from inner modules to outer modules
