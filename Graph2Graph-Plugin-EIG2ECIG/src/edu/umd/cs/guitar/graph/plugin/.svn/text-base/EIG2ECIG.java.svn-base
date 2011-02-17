/*  
 *  Copyright (c) 2009-@year@. The  GUITAR group  at the University of
 *  Maryland. Names of owners of this group may be obtained by sending
 *  an e-mail to atif@cs.umd.edu
 * 
 *  Permission is hereby granted, free of charge, to any person obtaining
 *  a copy of this software and associated documentation files
 *  (the "Software"), to deal in the Software without restriction,
 *  including without limitation  the rights to use, copy, modify, merge,
 *  publish,  distribute, sublicense, and/or sell copies of the Software,
 *  and to  permit persons  to whom  the Software  is furnished to do so,
 *  subject to the following conditions:
 * 
 *  The above copyright notice and this permission notice shall be included
 *  in all copies or substantial portions of the Software.
 *
 *  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
 *  OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
 *  MERCHANTABILITY,  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
 *  IN NO  EVENT SHALL THE  AUTHORS OR COPYRIGHT  HOLDERS BE LIABLE FOR ANY
 *  CLAIM, DAMAGES OR  OTHER LIABILITY,  WHETHER IN AN  ACTION OF CONTRACT,
 *  TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
 *  SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE. 
 */
package edu.umd.cs.guitar.graph.plugin;

import java.awt.Event;

import java.io.FileReader;
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.IOException;

import java.util.ArrayList;
import java.util.Hashtable;
import java.util.LinkedList;
import java.util.List;
import java.util.Stack;
import java.util.Vector;

import java.util.StringTokenizer;
import org.apache.log4j.Level;


import edu.umd.cs.guitar.model.GUITARConstants;
import edu.umd.cs.guitar.model.data.EFG;
import edu.umd.cs.guitar.model.data.EdgeMappingListType;
import edu.umd.cs.guitar.model.data.EdgeMappingType;
import edu.umd.cs.guitar.model.data.EventGraphType;
import edu.umd.cs.guitar.model.data.EventType;
import edu.umd.cs.guitar.model.data.EventsType;
import edu.umd.cs.guitar.model.data.InitialMappingListType;
import edu.umd.cs.guitar.model.data.InitialMappingType;
import edu.umd.cs.guitar.model.data.MappingType;
import edu.umd.cs.guitar.model.data.PathType;
import edu.umd.cs.guitar.model.data.RowType;
import edu.umd.cs.guitar.model.wrapper.EventWrapper;
import edu.umd.cs.guitar.util.GUITARLog;


/**
 * @author <a href="mailto:baonn@cs.umd.edu"> Bao N. Nguyen </a>
 * 
 */
public class EIG2ECIG extends G2GPlugin {

   /**
    * Map of <event, all events used to reveal it> Note that we
    * don't store all predecessor events here
    */
   Hashtable<EventType, Vector<EventType>> preds;

   /**
    * Map of <event, all successor events>
    */
   Hashtable<EventType, Vector<EventType>> succs;

   List<EventType> initialEvents = null;

   /**
    * @param inputGraph
    */
   public EIG2ECIG(EFG inputGraph) {
      super(inputGraph);
   }

   /*
    * (non-Javadoc)
    * 
    * @see
    * edu.umd.cs.guitar.graph.plugin.G2GPlugin#parseGraph(edu.umd.cs.guitar
    * .model.data.EFG)
    */
   @Override
   public boolean parseGraph() {
      int edgeTotal      = 0;
      int eventNotFound  = 0;

      int edgeInputGraph = 0;
      int edgeAdded      = 0;

      boolean result     = true;

      String ecigFilename = "/tmp/ecig.txt";

      List<String[]> eventData = getEventInteractionData(ecigFilename);

      outputGraph = factory.createEFG();

      EventsType eventList      = factory.createEventsType();
      EventGraphType eventGraph = factory.createEventGraphType();

      // Select interaction events
      List<EventType> lEventList = getEventList();

      /*
       * Validate input ECIG data against event list from input graph.
       * Each event from the input event pair must exist in the input
       * event graph.
       */
      for (String[] strArray : eventData) {
         boolean found0 = false, found1 = false;   
   
         for (EventType event : lEventList) {
            if (strArray[0].equals(event.getEventId())) {
               found0 = true;
            }
            if (strArray[1].equals(event.getEventId())) {
               found1 = true;
            }
         }

         if (found0 == false || found1 == false) {
            GUITARLog.Error("Event " + strArray[0] + " or "
                            + strArray[1] + " not found in input graph");
            result = false;
            eventNotFound++;
         }

         edgeTotal++;
      }

      // Exit if validation failed
      if (result == false) {
         GUITARLog.Error("Input ECIG data : " +
                        eventNotFound + " bad edges of " +
                        edgeTotal + " edges from ECIG data");

        return result;
      }

      // Set output set of events same as input set of events
      eventList.setEvent(lEventList);
      outputGraph.setEvents(eventList);

      /*
       * Build new edge list from event code interaction data.
       */
      List<RowType> lRowList = new ArrayList<RowType>();

      for (EventType firstEvent : lEventList) {
         int indexFirst = lEventList.indexOf(firstEvent);

         RowType row = factory.createRowType();
 
         for (EventType secondEvent : lEventList) {
            int indexSecond = lEventList.indexOf(secondEvent);
 
            int newCellValue = 0;

            int curCellValue = this.inputGraph.getEventGraph().getRow().
                               get(indexFirst).getE().get(indexSecond);

            // Count # edges in input graph
            if (curCellValue == 1) {
               edgeInputGraph++;
            }

            for (String[] strArray : eventData) {

               // The edge has been found in the ECIG data
               if (!strArray[0].equals(firstEvent.getEventId()) ||
                   !strArray[1].equals(secondEvent.getEventId())) {
                  continue;
               }

               newCellValue = 1;

               GUITARLog.Debug("Adding edge:" + firstEvent.getEventId() +
                               " -> " + secondEvent.getEventId());

               // Existing EFG / EIG must have this edge
               if (curCellValue == 0) {
                  GUITARLog.Error("Edge " +
                                  indexFirst + " -> " + indexSecond +
                                  " from ecig data not found in EIG ");
                  result = false;
               } else {
                  edgeAdded++;
               }
            }

            row.getE().add(indexSecond, newCellValue);
         }

         lRowList.add(indexFirst, row);
      }

      // Set output value
      eventGraph.setRow(lRowList);
      outputGraph.setEventGraph(eventGraph);

      // Stats / logs
      GUITARLog.Info("Input  graph    : " + edgeInputGraph + " edges");
      GUITARLog.Info("Output graph    : " + edgeAdded + " of " + edgeTotal +
                     " ECIG edges from ECIG data");

      return result;
   }

   /**
    * Get event list of interest. For example System interaction and Terminal
    * events in EIG
    * 
    * @return
    */
   List<EventType> getEventList() {

      List<EventType> lOutputEvents = new ArrayList<EventType>();

      if (this.inputGraph == null) {
         return lOutputEvents;
      }

      List<EventType> lInputEvents = inputGraph.getEvents().getEvent();

      for (EventType event : lInputEvents) {
         if (GUITARConstants.SYSTEM_INTERACTION.equals(event.getType()) ||
             GUITARConstants.TERMINAL.equals(event.getType())) {
            lOutputEvents.add(event);
         }
      }

      return lOutputEvents;
   }


   /**
    * Read a file contain comma-separated event code interaction event pairs.
    *
    * @return
    */
   List<String[]> getEventInteractionData(String filename) {

      List<String[]> list = new ArrayList<String[]>();

      try {
         BufferedReader br = new BufferedReader( new FileReader(filename));

         String string = null;
         StringTokenizer st = null;
 
         while( (string = br.readLine()) != null) {
            int      i = 0;
            String[] e = new String[2];

            // Split comma separated line using ","
            st = new StringTokenizer(string, " ");

            // Although a while is used, we expect 2 tokens 
            while(st.hasMoreTokens()) {
               e[i++] = "e" + st.nextToken();
            }

            // Only two tokens expected
            if (e[0] != null && e[1] != null) {
               list.add(e);
            }
         }
      } catch (FileNotFoundException e) {
         e.printStackTrace();
      } catch (IOException e) {
         e.printStackTrace();
      }

      return list;
   }


   // End of class
}
