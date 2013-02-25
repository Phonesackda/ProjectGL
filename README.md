ProjectGL
=========

#include <stdlib.h>
#include <stdio.h>
#include <math.h>
#ifdef __APPLE__
#include <GLUT/glut.h>
#else
#include <GL/glut.h>
#endif

#define ROT_INC  	0.1

/* forward declare the default drawing function */
void drawSphere(void);

/*
 * local static variables: g_rotate used to keep track of global rotation
 * g_rotInc is the rotation increment each frame
 *
 */
static GLfloat g_rotate = 0;
static GLfloat g_rotInc = ROT_INC; /* degree increment for rotation animation */

/* function pointer to a function called by display to draw the geometry */
static void (*drawPrimP)(void) = drawSphere;

/*
 * basic geometry drawing functions - just wrappers to the glut
 * wire drawing functions
 */
void drawSphere(void) {
	glutWireSphere(6.0,20,20);
}

void drawCube(void) {
	glutWireCube(6.0);
}

void drawCone(void) {
	glutWireCone(6.0, 8.0, 10, 20);
}

void drawTorus(void) {
	glutWireTorus(1.0, 6.0, 10, 20);
}

void drawIcos(void) {
	glPushMatrix();
	glScalef(6.0,6.0,6.0);
	glutWireIcosahedron();
	glPopMatrix();
}

void drawTeapot(void) {
	glutWireTeapot(6.0);
}

/*
 * setPrim
 *
 * callback, called by the GLUT when the user selects a menu option.
 * This function currently contains "magic numbers" in that the menu numbers
 * and their functions are hard coded. A better option would be to use macros
 * or have a data structure that relates menu number, name and function...
 */
void setPrim(int value) {
	switch(value) {
	case 1:
		drawPrimP = drawSphere;
		break;
	case 2:
		drawPrimP = drawCube;
		break;
	case 3:
		drawPrimP = drawCone;
		break;
	case 4:
		drawPrimP = drawTorus;
		break;
	case 5:
		drawPrimP = drawIcos;
		break;
	case 6:
		drawPrimP = drawTeapot;
		break;
	default:
		fprintf(stderr, "3dPrim: unknown menu option %d\n", value);
		break;
	}
}

/*
 * display
 *
 * This function is called by the GLUT to display the graphics
 *
 */
void display(void)
{
    glClear( GL_COLOR_BUFFER_BIT );

	/* set matrix mode to modelview */
    glMatrixMode(GL_MODELVIEW);
	/* save matrix */
	glPushMatrix();

	/* global rotation */
	glRotatef(g_rotate,1.0,1.0,1.0);

	/* draw the geometry */
	(*drawPrimP)();

	/* restore matrix */
	glPopMatrix();

	/* swap buffers to display the frame */
 	glutSwapBuffers();
}


/*
 * myReshape
 *
 * This function is called whenever the user (or OS) reshapes the
 * OpenGL window. The GLUT sends the new window dimensions (x,y)
 *
 */
void myReshape(int w, int h)
{
	/* set viewport to new width and height */
	/* note that this command does not change the CTM */
    glViewport(0, 0, w, h);

	/* 
	 * set viewing window using perspective projection
	 */
    glMatrixMode(GL_PROJECTION); 
    glLoadIdentity(); /* init projection matrix */

	/* perspective parameters: field of view, aspect, near clip, far clip */
	gluPerspective( 60.0, (GLdouble)w/(GLdouble)h, 0.1, 40.0);

	/* set matrix mode to modelview */
    glMatrixMode(GL_MODELVIEW);
	glLoadIdentity();
	gluLookAt(0.0, 0.0, 20.0, /* eye at (0,0,20) */
			  0.0, 0.0, 0.0, /* lookat point */
			  0.0, 1.0, 0.0); /* up is in +ive y */
}

/*
 * myKey
 *
 * responds to key presses from the user
 */
void myKey(unsigned char k, int x, int y)
{
	switch (k) {
		case 'q':
		case 'Q':	exit(0);
		break;
	default:
		printf("Unknown keyboard command \'%c\'.\n", k);
		break;
	}
}


/*
 * myMouse
 *
 * function called by the GLUT when the user presses a mouse button
 *
 * Here we increment the global rotation rate with each press - left to do a
 * positive increment, right for negative, middle to reset
 */
void myMouse(int btn, int state, int x, int y)
{   

    if(btn==GLUT_LEFT_BUTTON && state == GLUT_DOWN) g_rotInc += ROT_INC;
	if(btn==GLUT_MIDDLE_BUTTON && state == GLUT_DOWN) g_rotInc = ROT_INC;
	/* if(btn==GLUT_RIGHT_BUTTON && state == GLUT_DOWN) g_rotInc -= ROT_INC;
	*/
	/* force redisplay */
	glutPostRedisplay();
}   

/*
 * myIdleFunc
 *
 * increments the rotation variable within glutMainLoop
 */
void myIdleFunc(void) {

	g_rotate += g_rotInc;

	/* force glut to call the display function */
	glutPostRedisplay();
}


/*
 * main
 *
 * Initialization and sets graphics callbacks
 *
 */
int main(int argc, char **argv)
{
	/* glutInit MUST be called before any other GLUT/OpenGL calls */
    glutInit(&argc, argv);

	/* set double buffering, note no z buffering */
    glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE);
    glutInitWindowSize(500, 500);
    glutCreateWindow("3D Shapes Test");

	/* set callback functions */
    glutReshapeFunc(myReshape);
    glutDisplayFunc(display);
	glutIdleFunc(myIdleFunc);
	glutKeyboardFunc(myKey);
	glutMouseFunc(myMouse);

	/* set up right menu */
	glutCreateMenu(setPrim);
	glutAddMenuEntry("Sphere", 1);
	glutAddMenuEntry("Cube", 2);
	glutAddMenuEntry("Cone", 3);
	glutAddMenuEntry("Torus", 4);
	glutAddMenuEntry("Icosahedron", 5);
	glutAddMenuEntry("Teapot", 6);
	glutAttachMenu(GLUT_RIGHT_BUTTON);

	/* set clear colour */
	glClearColor(1.0, 1.0, 1.0, 1.0);

	/* set current colour to black */
	glColor3f(0.0, 0.0, 0.0);
	
    glutMainLoop();
	 
	return 0;
}
