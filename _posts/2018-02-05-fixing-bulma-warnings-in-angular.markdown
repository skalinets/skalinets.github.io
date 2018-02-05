
During last few months I dived into frontend development. Well, not frontend but rather full-stack.
My current stack is ASP.NET Core C# web backend site + angular.io client part. It appeared to be a 
nice mix, very console friendly and I had a lot of fun working with it.

To be able to quickly generate descent looking web pages without getting trapped into css hell developers
use very clever trick: CSS frameworks. For years the leader here has been (and seems to remain) Twitter's 
[bootstrap](https://getbootstrap.com/). But this time I decided to try something else and chose 
[Bulma](https://mangelmaxime.github.io/Fulma/). It is nice and lightweight and rather modern so why not
to give it a try?

The process of adding Bulma to the angular application is straightforward. First we add bulma package via
[yarn](https://yarnpkg.com/): 
{% highlight bash %}
yarn add bulma
{% endhighlight%}

and then add reference to it fo `angular-cli.json`:
{% highlight json %}
      "styles": [
        "../node_modules/bulma/bulma.sass",
        "styles.scss"
      ],
{% endhighlight%}

Thats all! But after running site with `ng serve` I've got a lot of console warnings: 

![Bulma Angular Warnings](/assets/angular-warnings.png)

Long story short, I started from this [issue](https://github.com/angular/angular-cli/issues/7991) and 
finally found solution [here](https://github.com/jgthms/bulma/issues/1190#issuecomment-356672849). And also
I discovered that instead of being added into `angular-cli.json`, styles can be just imported into your
`styles.scss':

{% highlight scss %}
$variable-columns: false;
@import '~bulma/bulma';
{% endhighlight %}

Now warnings have gone and we can continue hacking our angular app :)