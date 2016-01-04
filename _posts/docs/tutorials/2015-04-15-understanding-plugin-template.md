---
title: Understanding the Plugin Template
categories: docs tutorials
layout: docpost
---

This is an explanation for the code found in the [plugin template module](/modules/plugin-template/) and a brief introduction to the DefaultGUI framework it abstracts. The plugin template is a starting point for building modules from scratch, but you can fork [any other module](/modules/), too. Below is the contents of `plugin-template.cpp` and `plugin-template.h`. Click on a function to see an explanation of what it does and how it relates to RTXI's framework. If anything is unclear, [let us know](https://github.com/rtxi/rtxi.github.io/issues).  

This tutorial will assume the reader has a basic understanding of C++, such as how to include libraries and source files, classes, inheritance, etc. For a quick run-down of C++ for people already familiar with programming, try [here](http://www.cplusplus.com/doc/tutorial/).  

####plugin-template.h
<div><a data-toggle="collapse" data-target="#header1">
{% highlight cpp %}
/*
* This is a template header file for a user modules derived from
* DefaultGUIModel with a custom GUI.
*/

#include <default_gui_model.h>
{% endhighlight %}
</a></div>

<div class="collapse" id="header1">
	<p><code>default_gui_model.h</code> is the header for the <code>DefaultGUIModel</code> class that <code>PluginTemplate</code> abstracts from. The file itself is stored in <code>include/default_gui_model.h</code> in the <a href="https://github.com/rtxi/rtxi">RTXI repository</a>. <code>DefaultGUI</code> is on its own a mechanism by which users can create their own plugins. <code>PluginTemplate</code> simply obscures some of the more arcane functions and provides a more simplified programming experience. 
	</p>
</div>

<div><a data-toggle="collapse" data-target="#header2">
{% highlight cpp %}
class PluginTemplate : public DefaultGUIModel {

   Q_OBJECT
{% endhighlight %}
</a></div>

<div class="collapse" id="header2">
	<p> The <code>PluginTemplate</code> abstracts from <code>DefaultGUIModel</code> to fit within RTXI's framework. To fit within the Qt framework, it uses the <code>Q_OBJECT</code> macro, which tells Qt's meta-object compiler to insert code into the class that implement signals and slots. Signals and slots are how different <code>QObjects</code> communicate with one another. One <code>QObject</code> sends a signal that is received by another <code>QObject</code>'s slot. The <code>Q_OBJECT</code> macro indicates that the class is to be treated as a <code>QObject</code>. 
	</p>
</div>

<div><a data-toggle="collapse" data-target="#header3">
{% highlight cpp %}

   public:

      PluginTemplate(void);
      virtual ~PluginTemplate(void);

      void execute(void);
      void customizeGUI(void);
{% endhighlight %}
</a></div>

<div class="collapse" id="header3">
	<p> The functions of the constructor and destructor should be obvious. 
	</p>
	<p> The <code>execute</code> function is used to run code within the real-time loop. In other words, whatever get's put in here will be run in real-time. Whatever is run here should be as time and memory-efficient as possible. After all, to run RTXI at high frequencies (40-50 kHz) the code must take microseconds to execute. 
	</p>
	<p> The <code>customizeGUI</code> function is used to add and edit UI elements on the display. By default, <code>DefaultGUI</code> creates widget display that shows all user-designated parameter and state variables. For an example of what this looks like, view the UI for the <a href="https://github.com/rtxi/neuron/">neuron module</a>. To do things like add buttons or extra windows, the <code>customizeGUI</code> function is needed.
	</p>
</div>

<div><a data-toggle="collapse" data-target="#header4">
{% highlight cpp %}

   protected:

      virtual void update(DefaultGUIModel::update_flags_t);

   private:

      double some_parameter;
      double some_state;
      double period;

		void initParameters();
{% endhighlight %}
</a></div>

<div class="collapse" id="header4">
	<p>The <code>update</code> function is used to inject custom code around events triggered within the UI. Definitions of what those states are are found in the source file. 
	</p>
	<p>The <code>initParameters</code> function can also be used to initialize parameters. It is called within the module constructor, which is defined in the source files, too. 
	</p>
</div>

<div><a data-toggle="collapse" data-target="#header5">
{% highlight cpp %}

   private slots:
   // these are custom functions that can also be connected
   // to events through the Qt API. they must be implemented
   // in plugin_template.cpp

      void aBttn_event(void);
      void bBttn_event(void);

};
{% endhighlight %}
</a></div>

<div class="collapse" id="header5">
	<p><code>private slots:</code> are part of the Qt signals and slots API. They are used to connect some state change in a widget, like pushing a button, to some function. For the <code>PluginTemplate</code>, the slots are used to connect the buttons in the UI to the function definitions of <code>aBttn_event</code> and <code>bBttn_event</code>. 
	</p>
</div>

####plugin-template.cpp
<div><a data-toggle="collapse" data-target="#source1">
{% highlight cpp %}
/*
* This is a template implementation file for a user module derived from
* DefaultGUIModel with a custom GUI.
*/

#include <plugin-template.h>
#include <main_window.h>
#include <iostream>
{% endhighlight %}
</a></div>

<div class="collapse" id="source1">
	<p><code>main_window.h</code> is the header file for the main RTXI application window. It's included here so that the module being created can reference the main window as it's parent. That way, the modules will be displayed within the main window. <code>iostream</code> is for writing things to standard output (i.e. the terminal). In the base code, <code>PluginTemplate</code> doesn't write to output, but you can change that if you'd like. Just remember to avoid writes within the <code>execute</code> function, or you'll crash RTXI. 
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source2">
{% highlight cpp %}
extern "C" Plugin::Object *createRTXIPlugin(void){
   return new PluginTemplate();
}
{% endhighlight %}
</a></div>

<div class="collapse" id="source2">
	<p><code>extern "C"</code> is implemented to prevent name-mangling by the C++ compiler. Normally, C++ uses name-mangling to allow many different definitions of an object with a same name. In other words, object names can be overloaded. Because this is a module, RTXI needs to load the dynamically-allocated module in memory. If the name gets mangled by the C++ compiler, then RTXI has to look for whatever that name was mangled into; however, RTXI cannot do this, meaning that it will be unable to find any name-mangled module. Therefore, we use <code>extern "C"</code>.
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source3">
{% highlight cpp %}

static DefaultGUIModel::variable_t vars[] = {
   { "GUI label", "Tooltip description", DefaultGUIModel::PARAMETER
      | DefaultGUIModel::DOUBLE, },
   { "A State", "Tooltip description", DefaultGUIModel::STATE, },
};

static size_t num_vars = sizeof(vars) / sizeof(DefaultGUIModel::variable_t);
{% endhighlight %}
</a></div>

<div class="collapse" id="source3">
	<p><code>vars[]</code> is an array that contains instances of the <code>variable_t</code> struct, which is defined in <code>DefaultGUIModel</code>. <code>vars[]</code> is used to pass variables to be the <code>DefaultGUIModel</code> constructor. In the base code for this module, there are two variables, <code>GUI label</code> and <code>A State</code>. Each is separately instantiated within <code>vars[]</code>.
	</p>

	<p>Each element added to <code>vars[]</code> must have at least three components:</p>
	<dl class="dl-horizontal">
		<dt>string name</dt>
			<dd>the label for the variable made visible in the GUI</dd>
		<dt>string description</dt>
			<dd>tooltip description displayed when the mouse hovers over the label</dd>
		<dt>flags_t flags</dt>
			<dd>the type for the variable, such as <code>STATE</code>, <code>PARAMETER</code>, <code>INPUT</code>, <code>OUTPUT</code>, or <code>COMMENT</code></dd>
	</dl>

	<p>Note that the strings are of type <code>std::string</code>. Qt has its own string implementation, called <code>QString</code>, so take care to not confuse ordinary C strings with them. Also note that <code>flags_t</code> type is defined in <code>io.h</code>. It is an unsigned long int used within RTXI for input and output processing. Simply put, it is a method RTXI uses to define different types for variables that represent different components to be displayed by the GUI and/or processed by the module.
	</p>

	<p>Additional information about the type parameter can be added by using the <code>|</code> operator. Note that <code>|</code> here represents a bitwise OR operator. What this additional information does is apply an RTXI-specific data type to a variable within <code>vars[]</code>. In the base code for GUI label, the variable is of type <code>PARAMETER</code>, and <code>| DOUBLE</code> means that the variable is a parameter of type <code>DOUBLE</code>, a data type that stores 8-byte floating point numbers. If we wanted integers, we would use <code>INTEGER</code> instead of <code>DOUBLE</code>. Again, note that <code>PARAMETER</code>, <code>INTEGER</code>, etc. are not native C++ types but instead are RTXI-specific. 
	</p>

	<p>The GUI is made by iterating through all the elements of <code>vars[]</code> and appending a widget to the GUI window for each one. Because the vars array does not have a native count of the number of elements within, we create a variable called <code>num_vars</code> that stores that number.
	</p>

	<p>The size of the elements within are contingent on their types, so the <code>variable_t</code> type is used within the <code>sizeof()</code> command. Because the count of variables cannot change after the module is compiled and run within RTXI, <code>num_vars</code> is declared as static, meaning that it cannot be changed from within the program after it is initialized.
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source4">
{% highlight cpp %}

PluginTemplate::PluginTemplate(void) : DefaultGUIModel("PluginTemplate with Custom GUI", ::vars, ::nu    m_vars) {
   setWhatsThis("<p><b>PluginTemplate:</b><br>QWhatsThis description.</p>");
   DefaultGUIModel::createGUI(vars, num_vars); // this is required to create the GUI
   customizeGUI();
	initParameters();
   update( INIT ); // this is optional, you may place initialization code directly into the construct    or
   refresh(); // this is required to update the GUI with parameter and state values
   QTimer::singleShot(0, this, SLOT(resizeMe()));
}

PluginTemplate::~PluginTemplate(void) { }
{% endhighlight %}
</a></div>

<div class="collapse" id="source4">
	<p><code>PluginTemplate</code> inherits from <code>DefaultGUIModel</code>., so when its constructor is called, so is that of <code>DefaultGUIModel</code>. <code>DefaultGUIModel</code>'s constructor takes three arguments:
	</p>
	<ol>
		<li>A string representing the name of the module displayed in its titlebar</li>
		<li>The <code>vars[]</code> array</li>
		<li><code>num_vars</code></li>
	</ol>
	<p>The functions called within the constructor are defined later. The <code>QTimer::singleShot</code> call is so make the UI resize properly if <code>customizeGUI</code> is used to edit the default UI.  
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source4-5">
{% highlight cpp %}

void PluginTemplate::initParameters(void) {
	some_parameter = 0;
	some_state = 0;
   return;
}
{% endhighlight %}
</a></div>

<div class="collapse" id="source4-5">
	<p>The <code>initParameters</code> function is an optional function you can use to initialize variables. Of course, you can define all this in the module constructor. Note that here, the variablles that are initialized are those that are ultimately linked to a <code>PARAMATER</code> and <code>STATE</code>. Therefore, <code>initParameters</code> should be called before <code>update(INIT)</code>.  
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source5">
{% highlight cpp %}

void PluginTemplate::execute(void) {
   return;
}
{% endhighlight %}
</a></div>

<div class="collapse" id="source5">
	<p>This is the execute loop. All the code in here is run using the real-time thread. Each cycle, RTXI takes all the code run in all definitions of various modules' <code>execute</code>. If you are outputting a signal in real-time, the code goes here. If you are reading in voltages to compute membrane resistance, etc., the code for that goes here. What <b>does not</b> go here is Qt code.</p>

	<p><b>Do not</b> operate on a Qt widget within the execute loop. Doing so forces the system to alter what is being displayed on the screen, which involves calls to the X server, video driver, the standard Linux kernel (if modesetting is active), etc., and all that has to be completed within the real-time period. Needless to say, that's an easy way to get knocked out of real-time. Don't run UI things within the execute loop. 
	</p>
	<p>Within the plugin template, nothing is running within the execute loop. You can add whatever code you need here, and it will run whenever the module is unpaused.  
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source6">
{% highlight cpp %}

void PluginTemplate::update(DefaultGUIModel::update_flags_t flag) {
   switch (flag) {
      case INIT:
         period = RT::System::getInstance()->getPeriod() * 1e-6; // ms
         setParameter("GUI label", some_parameter);
         setState("A State", some_state);
         break;

      case MODIFY:
         some_parameter = getParameter("GUI label").toDouble();
         break;

      case UNPAUSE:
         break;

      case PAUSE:
         break;

      case PERIOD:
         period = RT::System::getInstance()->getPeriod() * 1e-6; // ms
         break;

      default:
         break;
   }
}
{% endhighlight %}
</a></div>

<div class="collapse" id="source6">
	<p>This is the <code>update</code> function. It injects code into events that allow modules to adjust to changes in the RTXI application. Whenever events, like pushing the pause button or changing the real-time period via the Control Panel, are triggered, their code includes calls to the <code>update</code> function and pass to it flags relating to the case in which the function is being called. The cases are:
	</p>
	<dl class="dl-horizontal">
		<dt><code>INIT</code></dt>
			<dd><p>Called whenever the module is being initialized, particularly in the constructor. Things to initialize, for instance, are the value for RTXI's real-time period and other variables derived from it. It is here, too, that you should set <code>PARAMETER</code>, <code>COMMENT</code>, and <code>STATE</code> variables. This is done via calls, respectively, to <code>setParameter</code> <code>setComment</code> <code>setState</code>.</p>
			</dd>
		<dt><code>MODIFY</code></dt>
			<dd><p><code>MODIFY</code> is used whenever the Modify button is pressed within the UI. It can be called elsewhere, too. It's the module designer's choice. In the base code for the plugin template, the <code>GUI label</code> parameter is displayed in the GUI as a label that says "GUI label" followed by a text box containing a floating point value that the user can modify by typing some new value in the text box. When a value is typed in and the user clicks the "Modify" button in the GUI, the <code>MODIFY</code> flag is passed to <code>update</code>, and the changed value is extracted, converted to <code>std::double</code>, and stored in <code>some_parameter</code>. </p>
			</dd>
		<dt><code>UNPAUSE</code></dt>
			<dd><p>Called when the module is unpaused. </p>
			</dd>
		<dt><code>PAUSE</code></dt>
			<dd><p>Called when the module is paused. </p>
			</dd>
		<dt><code>PERIOD</code></dt>
			<dd><p>Called whenever the real-time period is changed by the Control Panel module. </p>
			</dd>
		<dt><code>EXIT</code></dt>
			<dd><p>Called whenever the module is closed. </p>
			</dd>
	</dl>
	<p>Look though our <a href="/doxygen">doxygen pages</a> to find out how to properly use built-in RTXI functions. 
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source7">
{% highlight cpp %}

void PluginTemplate::customizeGUI(void) {
   QGridLayout *customlayout = DefaultGUIModel::getLayout();

   QGroupBox *button_group = new QGroupBox;

   QPushButton *abutton = new QPushButton("Button A");
   QPushButton *bbutton = new QPushButton("Button B");
   QHBoxLayout *button_layout = new QHBoxLayout;
   button_group->setLayout(button_layout);
   button_layout->addWidget(abutton);
   button_layout->addWidget(bbutton);
   QObject::connect(abutton, SIGNAL(clicked()), this, SLOT(aBttn_event()));
   QObject::connect(bbutton, SIGNAL(clicked()), this, SLOT(bBttn_event()));

   customlayout->addWidget(button_group, 0, 0);
   setLayout(customlayout);
}
{% endhighlight %}
</a></div>

<div class="collapse" id="source7">
	<p>The <code>customizeGUI</code> function is designed for users to customize the default UI output created by the <code>DefaultGUIModel</code> class. The initial module is created by the <code>createGUI</code> function called above in the constructor. Broadly speaking, this function takes the output of <code>createGUI</code> and adds custom elements to it. To use this function, you will need to know what the Qt classes are and how to use them properly. Documentation is available on Nokia's website. (<a href="http://doc.qt.io/qt-4.8/">http://doc.qt.io/qt-4.8/</a>)
	</p>
	<p>To customize the module UI, it first needs to be grabbed and stored in a variable, which is accomplished in the first line of the function. <code>DefaultGUIModel::getLayout()</code> returns an object of type <code>QGridLayout</code>. This essentially is a widget that allows Qt objects to be aligned according to a grid layout. The layout of <code>DefaultGUIModel</code> is as follows:  
	</p>
	<img class="img-responsive" src="/assets/img/tutorials/default_gui_layout.svg" style="max-height:450px;"></img>
	<br>
	<p>By default, the main UI is displayed within the grid at coordinates (1,0) and the utility box with the pause, modify, and exit buttons at coordinates (10,0). All other positions on the grid are empty.</p>
	<p>Note that the size of grid coordinates are arbitrary. They are as big as the widgets that are stuck within them. They're equally-sized in the diagram just for clarity. Look through Qt's documentation for instructions on how to set widgets at specific positions in the grid layout. 
	</p>
	<p>Also, note that coordinates that are empty are not rendered, so there is no big gap between the Main UI and the Button UI because there is nothing put in spots (2,0) to (9,0). Keep in mind, though, that the <code>QGridLayout</code> will not allow objects in diffrent rows to overlap. For example, if you added an object to (1,5), the UI in (0,1) would render, followed by a gap the size of the widget in (1,5), and then the buttons in (0,10). The object in (1,5) would be shown in column 1. 
	</p>

	<p>The rest of the code in <code>customizeGUI</code> creates two buttons and sticks them into coordinate (0,0). The buttons are then connected to functions using Qt's signals and slots API. When a button is pressed, it emits a signal that it has been pressed, and the <code>connect</code> connects that signal to a function. The effect is then to call the function whenever the button is pressed.
	</p>

	<p>When all the modifications are complete, they need to be added to the module layout that was grabbed by <code>DefaultGUIModel::getLayout()</code>. This is done using the <code>setLayout</code> command.
	</p>
</div>

<div><a data-toggle="collapse" data-target="#source8">
{% highlight cpp %}

// functions designated as Qt slots are implemented as regular C++ functions
void PluginTemplate::aBttn_event(void) { }

void PluginTemplate::bBttn_event(void) { }
{% endhighlight %}
</a></div>

<div class="collapse" id="source8">
	<p>These functions are called whenever their respective buttons are pressed. By default, the functions don't do anything. It is up to users to add what they want. 
	</p>
</div>