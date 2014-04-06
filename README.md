Clusterer
=========

Clustering library for points of interest in Android v2 Maps.

Getting Started
---------------

You can add the project with maven or gradle via adding the appropiate dependence.

```groovy
 compile 'io.nlopez.clusterer:library:1.0.2'
```

Preparing your app
------------------

You should have a working Google Maps v2 (the one from Google Play Services 13+) in your application, with the proper keys and all the stuff needed. Then you should follow these steps.

### In your Model

In the model you are using for referencing a point of interest, you should implement the `Clusterable` interface. That interface only needs a method that returns a LatLng position with the appropiate coordinates, called `getPosition()`.

An example:
```java
public class MyPoi implements Clusterable {
	private String name;
	private LatLng position;

	public MyPoi(String name, LatLng position) {
		this.name = name;
		this.position = position;
	}

	public String getName() { return name; }

	@Override
	public LatLng getPosition() { return position; }
}
```

### Where you have a reference to the GoogleMap object

In the class you are using the GoogleMap object, you must instance the **Clusterer** object.

```java
	GoogleMap map;
	Clusterer clusterer;

	// [...]

	private void initClusterer() {
		clustered = new Clusterer(this, map);
	}

```

The **Clusterer** will control your zoom and will reload the cluster markers when the zoom changes without your interaction.

**NOTE** You shouldn't control the `GoogleMap.OnCameraChangeListener` on your own anymore, you should let `Clusterer` handle it. If you want to add some functionality to it or control on your own, you should set that callback in `Clusterer` instead.

Example:

```java
    clusterer.setOnCameraChangeListener(
        new GoogleMap.OnCameraChangeListener() {
            @Override
            public void onCameraChange(CameraPosition cameraPosition) {
                // Do your thing in here
            });
        }
    );
```

Customization
-------------

You can customize the marker's appearance, even if it's a cluster marker. You have **two listeners** that will be invoked when the markers are being generated by the **Clusterer** and will be able to provide the markers with the **MarkerOptions** data we want.

Here you have an example that displays the value of an attribute called `description` from our `PointOfInterest` model as the subtitle, and an attribute `name` as the title. If the marker is a cluster, it will display a drawable with the number of points contained in the cluster painted on top of it.

```java
		clusterer.setOnPaintingMarkerListener(new OnPaintingClusterableMarkerListener() {

			@Override
			public void onMarkerCreated(Marker marker, Clusterable clusterable) {
                // If you want to perform some animation or whatever, you could do it here
			}

			@Override
			public MarkerOptions onCreateMarkerOptions(Clusterable clusterable) {
				PointOfInterest poi = (PointOfInterest) clusterable;
				return new MarkerOptions().position(clusterable.getPosition()).title(poi.getName()).snippet(poi.getDescription());
			}
		});

		clusterer.setOnPaintingClusterListener(new OnPaintingClusterListener() {

			@Override
			public void onMarkerCreated(Marker marker, Cluster cluster) {
                // If you want to perform some animation or whatever, you could do it here
			}

			@Override
			public MarkerOptions onCreateClusterMarkerOptions(Cluster cluster) {
				return new MarkerOptions()
						.title("Clustering " + cluster.getWeight() + " items")
						.position(cluster.getCenter())
						.icon(BitmapDescriptorFactory.fromBitmap(
						        getClusteredLabel(cluster.getWeight(),
								MainActivity.this)));
			}
		});

	}

    // A little method that creates a composition with an image from the drawables and the number of pois included in a particular cluster
    private Bitmap getClusteredLabel(Integer count, Context ctx) {
        Resources r = ctx.getResources();
        Bitmap res = BitmapFactory.decodeResource(r, R.drawable.circle_red);
        res = res.copy(Bitmap.Config.ARGB_8888, true);
        Canvas c = new Canvas(res);

        Paint textPaint = new Paint();
        textPaint.setAntiAlias(true);
        textPaint.setTextAlign(Paint.Align.CENTER);
        textPaint.setTypeface(Typeface.DEFAULT_BOLD);
        textPaint.setColor(Color.WHITE);
        float density = getResources().getDisplayMetrics().density;
        textPaint.setTextSize(16 * density);

        c.drawText(String.valueOf(count.toString()), res.getWidth() / 2, res.getHeight() / 2 + textPaint.getTextSize() / 3, textPaint);

        return res;
    }
```

As you can see, the listeners for the interfaces `OnPaintingClusterableMarkerListener` and `OnPaintingClusterListener` let you do anything you want to the markers, whether if they are a cluster or a point of interest in their own.

You can take a look to the provided [sample application](sample/).

License
-------

The MIT License (MIT)

Copyright (c) 2013 Nacho Lopez

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
