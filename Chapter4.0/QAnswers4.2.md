Q1.

The <code>.link()</code> function effectively returns a Reference to a Resouce previously saved in <code>/storage/</code>. Conceptually this consists in creating a pointer, or capability, from a Resource in <code>/storage/</code> to the <code>/public/</code> or <code>/private/</code> areas.

Q2.

Resource Interfaces can be used to establish an access framework to the Resource in question. Since the exposure of Resources in the <code>/public/</code> path is done via the <code>.link()</code> function and this one accepts a Resource Interface as well as a simple Resource as target, Interfaces can/should be used to control the access to the Resource fields. 
