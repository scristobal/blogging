# blogging

A repo for my blog posts

## Requirements

-   **storage**: unique source + client synchronization + version control
-   **edition**: versatility to edit on the go (almost) anywhere
-   **publication**: fully automated + push to different sources

### Storage

Use [GitHub](https://github.com)

### Edition

If you like it simple and beauty: Panda (only Mac/iOS) [Alpha](https://bear.app/alpha/)

> use `$ open -a panda file.md` to open the editor might want to create an alias on `.zshrc` with `alias panda="open -a panda"`

Similarly [Obsidian](https://www.obsidian.md) although Panda's live preview is more mature, Obsidian has more features in the form of [plugins](https://obsidian.md/plugins#):

-   [Git](https://github.com/denolehov/obsidian-git)
-   [Code block copy](https://github.com/jdbrice/obsidian-code-block-copy)
-   [Vale](https://github.com/marcusolsson/obsidian-vale) a sort of linting
-   [Kroki](https://github.com/gregzuro/obsidian-kroki) render diagrams
-   [Reading-time](https://github.com/avr/obsidian-reading-time)

Another markdown editor, [InkDrop](www.inkdrop.app), haven't tried it yet. Not sure wether it can work on top of github or it uses its own could sync.

Online edition with [Marked](www.marked.js.org)

A more focused alternative `nvim` [NeoVim](https://neovim.io/) that also runs on older devices

### Publication

-   [GitHub pages](https://pages.github.com/) near zero setup
-   [Netlify](http://netlify.com) + [Eleventy](https://www.11ty.dev/) a bit more work and versatility
-   [Next.js on Vercel](https://vercel.com/solutions/nextjs) same
-   distribution to 3rd party (eg. medium.com, dev.to, etc.) using GitHub actions?
