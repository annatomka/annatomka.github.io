Once upon a time there was a request from the customer to be able to draw polygons on a map. Later there were more and more feature requests to transfom this polygon editor to a swiss army knife. One of them was to prevent users to draw concave polygons on the map.

This would be pretty easy - I thought - if it were only a wonderful Openlayers function e.g <code>polygon.isConvex()</code>. But it wasn't, or at least I could find it in version 2.13.1.
I was searching the internet, trying to find the simplest solution, because I was sure there is a more elegant and shorter solution than hacking with angles.

Finally stackoverflow came to my help (as always :). In <a href="http://stackoverflow.com/questions/471962/how-do-determine-if-a-polygon-is-complex-convex-nonconvex">this thread</a> I found the most suitable algorithm for my need.


> A polygon is a set of points in a list where the consecutive points form the boundary. It is much easier to figure out whether a polygon is convex or not (and you don't have to calculate any angles, either):</div>

For each consecutive pair of edges of the polygon (each triplet of points), compute the z-component of the cross product of the vectors defined by the edges pointing towards the points in increasing order. Take the cross product of these vectors:</div>
<pre><code> given p[k], p[k+1], p[k+2] each with coordinates x, y:
 dx1 = x[k+1]-x[k]
 dy1 = y[k+1]-y[k]
 dx2 = x[k+2]-x[k+1]
 dy2 = y[k+2]-y[k+1]
 zcrossproduct = dx1*dy2 - dy1*dx2
</code></pre>
<div>
The polygon is convex if the z-components of the cross products are either all positive or all negative. Otherwise the polygon is nonconvex.</div>
<div>
If there are N points, make sure you calculate N cross products, e.g. be sure to use the triplets (p[N-2],p[N-1],p[0]) and (p[N-1],p[0],p[1]).</div>
</blockquote>
<br />
You can checkout my Javascript implementation of this algorithm <a href="https://gist.github.com/annatomka/82715127b74473859054">here</a>.<br />
<a style="margin-left: auto; margin-right: auto;"><img border="0" src="http://1.bp.blogspot.com/-wAR9htveRgk/VdB0wUYMt8I/AAAAAAAAIDI/MiROVlD0zt0/s1600/geofence_invalid_concave2.png" /></a>
Openlayers concave shape example
